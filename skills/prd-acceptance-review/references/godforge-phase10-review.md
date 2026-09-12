# Godforge Phase 10 Fresh Review Notes

Use this reference when auditing replay, post-match, spectator, streamer mode, and history acceptance in Godforge-like API/game-server systems.

## Fail-closed checks

- **Deterministic replay:** a `resimulate(seed, inputs, events)` helper is not evidence of determinism when the seed/initial snapshot are unused or when all stored events are passed back as `terminalEvents`. Verify that the reducer can reconstruct the result from the canonical input stream and that a regression still passes if the stored derived event list is removed or altered.
- **Production replay wiring:** trace event and input producers outside tests. Ranked allocation capturing seed/config and ranked result recording `match_finished` proves only metadata/terminal wiring; it does not prove that the live game server sends gameplay inputs/events into the replay service.
- **Authoritative post-match:** verify the ranked terminal path derives winner/final score/stats from the authoritative live match after downstream release. Treat an admin POST accepting arbitrary full stats as an ingestion adapter, not proof that the game server produces those stats.
- **Ranked spectator privacy:** never trust `body.ranked` or a similar client flag. Resolve ranked status from the authoritative match/allocation registry. A returned `{delaySeconds, sensitive:false}` policy is declarative only until the actual feed/transport enforces delay and redaction.
- **Streamer mode:** trace the per-actor toggle into the real response or WebSocket frame path. A stored Set/Map plus an unused `view()` helper is decorative and fails acceptance.
- **History:** check actor scoping and restart behavior. A global in-memory key list can satisfy a same-process happy path while leaking every user's match history and losing it on restart.

## Scope distinction

Report durable replay/post-match persistence and live spectator transport separately when the phase only implements in-memory services or a join-policy contract. Do not convert those deferred concerns into a false pass, but do not confuse them with a current blocker unless the phase acceptance explicitly requires them.

## Minimal evidence pattern

- Search all non-test callers of `recordInput`, `recordEvent`, `view`, and post-match producers.
- Read the route and service together; service-level tests are insufficient for route trust-boundary controls.
- Run focused API/game-server/testing suites plus build, typecheck, lint, and format checks.
- Return structured JSON with `passed`, `blockers`, `missing_acceptance`, `fixes`, and `evidence`; no edits during a review-only pass.
