# Browser allocation and reload-reconnect seam

Use when a browser UI already has WebSocket handling but still receives match identity from build-time environment variables.

## Minimal production path

1. Browser creates or restores an API identity, persisting the opaque recovery credential so retries do not mint ghost guests.
2. Browser queues through the authenticated API and polls matchmaking with a bounded timeout. On timeout/error/unmount before allocation, cancel/dequeue best-effort.
3. The API mints a distinct opaque `join_token` per participant and sends the full `player_id → join_token` map to the game server through the protected allocation route.
4. The game server requires exact `player_ready.player_id + join_token` equality before consuming a slot or issuing a reconnect session token. A roster allowlist alone is insufficient because either participant can claim the peer ID.
5. Return only the requesting actor's join token. Cache allocation metadata per participant so either browser can retrieve the same allocation after the first participant triggers it; polling must not re-enqueue an allocated player.
6. Validate live API tickets fail-closed: non-empty match ID, `ws://` or `wss://` URL, exactly two distinct roster IDs, current API player included, and non-empty join token. Validate restored `sessionStorage` tickets separately; never cast parsed JSON or combine partial stored fields with environment fallbacks.
7. Derive civilization/slot only from authoritative roster position; never select the first returned player unconditionally.
8. Feed the API ticket into the existing WebSocket transport rather than creating a second path.
9. Store the validated ticket, WebSocket session token, and **next sequence number** in tab-scoped `sessionStorage`, keyed by match/player.
10. Display connection state from actual socket acknowledgement, not allocation success. Restore Play only after accepted reconnect/ack and an authoritative snapshot.

## Sequence invariant

A page reload resets React refs. If sequence restarts at `1`, an otherwise valid reconnect is rejected as duplicate/stale by the authoritative server. Persist the next sequence immediately whenever an envelope is emitted:

- initial ready sends `1`, stores next=`2`;
- reload reconnect sends `2`, stores next=`3`;
- subsequent power intents continue from `3`.

Every outbound path (ready, reconnect, power, ping, surrender) must allocate and persist sequence through one shared operation. Malformed, zero, negative, fractional, or nonnumeric stored values fall back to `1`.

## TDD checks

- malformed guest identity fails closed;
- pending matchmaking returns `null`;
- malformed or foreign ticket fails closed;
- stored next sequence survives reload;
- corrupt stored sequence falls back safely.

## Live verification

Start fresh isolated API/game-server/web processes on known-free ports with matching local dummy control tokens. Queue one real opponent through the same API, then prove:

- Home → Lobby → Find Match reaches Play;
- browser ticket match ID equals game-server metrics match ID;
- initial connection stores token and next sequence;
- full page reload returns to Play with the same match/player and increments sequence;
- only one active socket remains for that player;
- browser console has zero errors.

Restart all three processes after transport edits. HMR and stale allocation state are invalid evidence. Build-time ticket variables may remain as an explicit developer override, but production browser acceptance requires the runtime API-issued path.