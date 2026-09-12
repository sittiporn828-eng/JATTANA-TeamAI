# Godforge Phase 5 Networking Reference

## Validated slice

The Godforge workspace uses:

- `apps/game-server/src/server.ts`: side-effect-free authoritative `GameServer` core.
- `apps/game-server/src/index.ts`: WebSocket/HTTP adapter exporting `startGameServer()`.
- `apps/game-server/src/main.ts`: process-only entrypoint that calls `startGameServer()`.
- `apps/game-server/src/game-server.test.ts`: unit plus real ephemeral-port WebSocket integration tests.
- `packages/network-protocol/src/index.ts`: Zod envelope/intent parsing and `createEnvelope()`.
- `@godforge/simulation`: authoritative competitive match and power APIs.

## Important pitfalls

- Do not auto-start the server from `index.ts`; Vitest workers can import it and unexpectedly bind port 4000. Keep startup in `main.ts` and point `dev` at `src/main.ts`.
- Build `@godforge/simulation` and `@godforge/network-protocol` before game-server tests when workspace packages resolve through `dist`.
- For an ephemeral integration server, set `GAME_SERVER_PORT=0`, await the HTTP `listening` event, use `http.address().port`, and always call `stop()`.
- A transport `power_event` broadcast is not proof that the power effect was applied. Route `use_power` through authoritative validation/effect APIs and broadcast only the accepted result/state.
- A session token created by `join()` is useless to network reconnect unless it is included in the actual `player_ready` acknowledgement. Test the wire message, not only the domain return value.
- Validate identity before `join()`; otherwise an empty `player_id` can consume a slot and create a trivial denial-of-service path.
- Keep an `activeConnections[playerId]` guard. On close, disconnect only if the closing socket is still the active socket; an old socket must not disconnect a replacement.
- A client-supplied player ID is not authentication. Initial join needs an API-issued match assignment/ticket or an equivalent trusted boundary; post-join session tokens do not solve initial impersonation.
- Add WebSocket `maxPayload`, timestamp freshness/future checks, and per-socket rate limits. A simulation power limiter does not protect ping, ready, reconnect, camera, or malformed-message abuse.
- A green full-repo gate does not prove Phase 5 complete. Run the complete network acceptance matrix and obtain a fresh independent review after the final edit; any subsequent edit invalidates that review.

## Acceptance matrix

- Two players in one match: test unique admission and snapshot player count.
- Same source of truth: assert both clients receive the same `match_id`, `server_tick`, and authoritative world/state.
- Power visibility: connect two clients, send a valid intent from A, assert B receives a server-derived accepted event and resulting authoritative state.
- Idempotency: resend the same sequence and assert rejection with no state transition.
- Reconnect: receive the join token over the wire, disconnect, tick during grace, reconnect with the token, assert no duplicate and current snapshot; reject invalid/expired tokens.
- Reconnect race: open a replacement socket and ensure the old socket close cannot disconnect the replacement.
- Continuity: assert server tick advances while disconnected.
- Invalid boundary: malformed JSON, wrong match, unknown type, empty identity, stale/future timestamp, oversized payload, rate limit, and disconnected intent must return an error envelope.
- Trust boundary: verify initial identity comes from API-issued match assignment/ticket, not only a client-provided `player_id`.
- 20 TPS: verify wall-clock tick cadence with bounded jitter and cleanup of timers.
