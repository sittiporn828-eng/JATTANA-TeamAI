# Release and observability evidence

Use this compact checklist when a fresh reviewer keeps finding gaps after green gates.

## Release binding

- Store `{serverUrl, matchId}` from allocation per player/match.
- Release with the returned `matchId`; never reconstruct it from player order.
- The HTTP release helper must throw on non-2xx responses.
- Delete allocation bindings only after release succeeds; retain them and audit failure for retry otherwise.
- On game-server release: close sockets, clear connection/session/active-player maps, reset authoritative state, then return `available`.
- Regression probe: allocator returns `custom-room-123` while players are `p1,p2`; assert release receives `custom-room-123`.

## Rehearsal order

1. Start the executable service entrypoints (`dist/main.js` for game-server, API entrypoint separately).
2. Check `/health`, `/ready`, `/metrics`, `/metrics/dashboard`.
3. Create two guest sessions, queue both, matchmaking, submit result, verify release.
4. Run authenticated load scenario; complete/release its match.
5. Restart services before a second E2E if the first scenario can leave in-memory state.

## Metrics/alerts

Expose counters with service/version labels for HTTP, DB errors, disconnects, matchmaking, payment recovery, rate-limit rejects, trace-bearing requests, server ticks/connections/lifecycle, and release failures. Dashboard evaluation must call an injected sink in tests and a configured webhook/dispatcher in deployment; alert names alone are not delivery evidence.

## Evidence boundaries

- A passing health/load smoke is not match lifecycle evidence.
- An opt-in scenario is not default acceptance evidence unless the release command enables it.
- A UI slider is not acceptance evidence unless a real render/audio path consumes it.
- A bundled script is not evidence until executed against built services and its output is captured.
