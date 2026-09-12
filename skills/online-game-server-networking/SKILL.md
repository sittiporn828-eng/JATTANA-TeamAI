---
name: online-game-server-networking
description: "Use for authoritative multiplayer WebSocket servers."
version: 1.0.0
metadata:
  hermes:
    tags: [multiplayer, websocket, networking, deterministic, tdd]
---

# Online Game Server & Networking

Use when connecting an existing deterministic simulation to multiple clients over WebSocket/HTTP.

## Workflow

1. Read the phase PRD, current network protocol, simulation API, and tests before coding.
2. Reuse the existing `WorldState`, `stepWorld`, competitive match APIs, and versioned config. Never create a parallel world model.
3. Keep the authoritative core importable without side effects. Put process startup in a separate `main.ts`; tests must not bind a port or leak timers.
4. Use strict TDD vertical slices:
   - RED: two players join one match; duplicate sequence is rejected.
   - GREEN: match registry and per-player sequence guard.
   - RED: disconnect does not stop ticks; reconnect restores the same player.
   - GREEN: grace-period lifecycle.
   - RED: WebSocket client receives snapshot and delta; invalid envelopes get errors.
   - GREEN: protocol parser, fixed 20 TPS loop, broadcast.
   - RED: Player A's accepted power is observed by Player B.
   - GREEN: authoritative power validation/effect and event broadcast.
5. Build producer workspaces before tests that import workspace `dist`.

## Trust boundary

- Client messages are intents only; the server validates match id, player identity, membership, connection state, sequence, status, target, power/loadout, and rate limits.
- Do not treat a client-supplied `player_id` as authentication. Initial join must come from an API-issued match assignment/ticket or an equivalent trusted boundary; a server-generated session token only protects the reconnect/ post-join path.
- Bind initial identity at allocation with a per-player opaque join credential, not only a participant allowlist. The API sends the full `player_id → join_token` map over the protected allocation route; the game server requires exact `player_ready.player_id + join_token` equality before consuming a slot. Return only the authenticated actor's token, and cache allocation metadata per participant so both browsers can retrieve an already-created match. See `references/browser-allocation-reconnect.md`.
- Enforce lifecycle before accepting any game intent: reject all WS messages with a `SERVER_NOT_RUNNING`-style code when the server is not allocated/`running`. A handler that checks only `match_id` is still open to play-before-allocate and post-release play (release resets the active match id).
- Do NOT let the API/result layer trust a client-declared outcome. If the authoritative simulation already computes a winner (e.g. `winnerCivilizationId`/`victoryType`), surface it from the game-server over an authenticated HTTP endpoint (e.g. `GET /result`) and have the API fetch-and-verify before settling rating/progression. Derive winner/loser from the server outcome; reject if the match is not finished (`MATCH_NOT_FINISHED`) or the outcome is unreachable (`OUTCOME_UNAVAILABLE`). Remove client-supplied `valid` as an input — it is server authority.
- Do NOT let a client jump persisted progression to an arbitrary value. A `Math.max` without an upper bound is forward-only but unbounded; cap to current+1 per call (or equivalent) so a client can only advance incrementally.
- Parse envelopes with the existing Zod schema. Reject malformed/unknown messages, wrong match ids, stale/future timestamps, missing player identity, oversized payloads, and disconnected intents.
- Treat allocation as a strict typed wire contract: do not coerce duration with `Number(...)`, default malformed/non-object room data to `{}`, or replace malformed bans with `[]`. Require the JSON types the API promised, apply the documented finite duration maximum at both transport and authoritative-core boundaries, and regression-test coercible values plus overflow-sized durations.
- Return the join-issued session token in the actual network acknowledgement; testing `GameServer.join()` directly is insufficient proof of reconnect usability.
- When a settlement/acquisition flow flips a lock (e.g. `beginSettlement` → `'settling'`), EVERY early-return rejection path after the lock must restore it (e.g. `restoreSettlement`) or the flow wedges permanently (players stuck busy, allocation leaks). Enumerate all `return`s after the acquire; the 409 fail-closed paths are exactly where this is most often missed. Prove with a retry-after-reject test.
- Keep sequence state scoped to the match/player lifecycle, not a module-global map keyed only by match id.
- Treat sequence updates as authenticated state mutations. For reconnect, check duplicate sequence without writing, validate the session token and reconnect lifecycle, then commit `lastSequence` only after acceptance. A rejected reconnect—especially one carrying the maximum allowed sequence—must not poison a later legitimate reconnect. Leave one RED→GREEN regression proving invalid-token max-sequence followed by a valid lower-sequence reconnect succeeds.
- Enforce socket binding before every non-join intent at the transport choke point. An unbound socket may attempt only initial `player_ready` or authenticated `reconnect`; reject `use_power`, `surrender`, `ping`, and camera/interest intents before authoritative handling, replay ingestion, acknowledgement, or broadcast. Bound sockets must always use the server-owned player id rather than `payload.player_id`.
- Track the active socket per player. A reconnect race must not allow two live sockets, and a stale old socket's `close` handler must not disconnect the replacement.
- Validate identity before calling `join()`. Empty/malformed identity must not consume a match slot or create a denial-of-service state.
- Broadcast only after authoritative validation and state transition. Send explicit error envelopes.
- Add transport `maxPayload`, per-socket message limits, and freshness checks; simulation-only power throttles do not protect ping, ready, reconnect, camera, or malformed-message paths.

## Runtime

- Use a fixed 20 TPS (`50ms`) server tick independent of client render FPS.
- A disconnected player remains in the match during a configurable grace period; the match continues ticking.
- Reconnect reattaches to the existing record and returns the current authoritative snapshot; never create a duplicate.
- Enforce 1v1/unique-civilization constraints at the boundary when the phase requires it.
- Stop all timers, WebSocket servers, and HTTP servers in integration tests.

## Verification

- Focused tests: join/dedupe, disconnect/reconnect, WebSocket snapshot/delta, invalid message, power propagation.
- Network-level acceptance must cover: token returned in join ack; reconnect with valid token; invalid token; second socket/old-close race; empty identity before join; oversized/stale/rate-limited messages; and API-issued match assignment if the phase claims authenticated matchmaking.
- For browser reload reconnect, persist the **next outbound sequence** alongside the session token under match/player scope. Reloading a valid token with sequence reset to `1` is still correctly rejected as duplicate. Route every browser intent through one sequence allocator; see `references/browser-allocation-reconnect.md`.
- For backend-driven browser notifications, trace the real server event through the client transport to one browser event/listener. Test all native permission states: `granted` uses the Notification API; `denied`, `default`, and missing API must still surface an in-app toast/status fallback and must not silently drop the event. Keep permission-request UI separate from event handling, and clean up the listener on unmount.
- Full repo: `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `git diff --check`.
- Inspect actual socket messages, not only class construction.
- Request a fresh independent fail-closed review **after the final edit**. Any edit after review invalidates that review; any missing acceptance item means report baseline/vertical slice, not complete.
- Do not call a local green gate "complete" when auth/matchmaking, network reconnect, rate limits, interest management, or authoritative effect propagation remain unverified.
- After a round of bulk `patch` edits across multiple test files, re-read/grep the edited files for corruption before declaring done. A patch that rewrites `authorization: Bearer ${token}` lines can silently mangle the `Bearer` token literal (tool redaction can collapse it to `***`), producing a syntax error that only surfaces at test-run time. Run the affected suite and grep for `***` in auth headers before handing off — never end a turn with a known syntax error pending.
- Do not commit/push unless the user explicitly asks.

See `references/godforge-phase5-networking.md` for the validated project-specific slice and boundaries.
See `references/godforge-trust-boundary-hardening.md` for the ranked-result authorization, allocate-time identity binding, and lifecycle-guard fix pattern (4 confirmed trust-boundary bugs closed).
See `references/cross-service-flow-rehearsal.md` for a real API→game-server HTTP/WS rehearsal, contract-drift integration-test pattern, and allocation-setting verification.
See `references/allocation-contract-and-rehearsal-hardening.md` for strict room/duration/ban validation, finite duration defense, and bounded WebSocket-open cleanup.
See `references/browser-allocation-reconnect.md` for API-issued per-player join credentials, symmetric allocation retrieval, stored-ticket validation, queue cleanup, and reload reconnect closure.
