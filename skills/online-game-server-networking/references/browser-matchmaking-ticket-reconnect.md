# Browser matchmaking ticket and reconnect closure

Use when a browser must complete Home → Lobby → allocation → authoritative WebSocket join → reload reconnect.

## Secure contract

1. The API authenticates the player and mints a distinct opaque `join_token` for each participant.
2. The authenticated allocation request sends the complete `player_id → join_token` map to the game server over the protected control-plane route.
3. The game server stores that map and requires exact `player_ready.player_id + join_token` equality before consuming a slot or issuing the reconnect session token.
4. The matchmaking response returns only the requesting actor’s join token. Never return the peer token.
5. Cache allocation metadata per participant so either browser can poll and retrieve its existing allocation after the first participant triggers allocation. Polling must not re-enqueue an already allocated player.
6. Validate both live API responses and restored `sessionStorage` tickets at runtime. Never cast parsed JSON to a ticket type or combine partial stored fields with environment fallbacks.
7. Require exactly two distinct roster IDs and ensure the requesting player belongs to the roster.
8. Persist the next transport sequence beside the reconnect session token. Reload must send a strictly newer reconnect sequence.
9. Persist only the guest recovery credential needed to restore the same account; do not create a fresh guest on each retry.
10. On timeout/error/unmount before allocation, cancel/dequeue best-effort so abandoned browser attempts do not leave ghost queue entries.

## Minimum regressions

- Participant B retrieves the same `match_id/server_url` after participant A allocates, but receives a different join token.
- An authorized player ID paired with the opponent’s token returns `JOIN_TOKEN_INVALID` and does not consume a slot.
- Duplicate rosters and malformed/stale stored tickets fail closed.
- Timeout/error invokes queue cancellation.
- Initial join persists next sequence `2`; reload reconnect consumes `2`, persists `3`, restores the same player/match/world, and leaves one active socket.

## Live rehearsal

Use fresh isolated API/game-server/web processes and local dummy control credentials. Queue a real second identity through the API, drive the first browser through Lobby, compare the API-issued match ID to game-server metrics, reload the page, and verify `CONNECTED`, same player/match, sequence advancement, one active connection, restored canvas, and zero browser errors. Restart all producers after changing the wire contract.
