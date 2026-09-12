# Godforge Phase 10 — Replay, Post-Match, Spectator & Match History

Closure reference for the Phase 10 class of work (deterministic replay + post-match + spectator).
PRD acceptance (5): seed+inputs resimulate; replay seek works; post-match stats match server;
spectator sends no sensitive real-time info in ranked; match history browsable.

## Domain shapes (all injectable services, `app.inject()` HTTP tests)

### `replay.ts` — ReplayService
- `capture(matchId, seed)` where seed = `{ worldSeed, rngSeed, simulationVersion, balanceVersion, mapVersion, initialSnapshot }` — the resimulation contract is seed bundle + event stream.
- `recordEvent(matchId, kind, tick, data)` — 9 event kinds: `power_used`, `player_disconnected`, `war_started`, `city_created`, `city_captured`, `capital_critical`, `capital_recovered`, `capital_eliminated`, `match_finished`.
- Checkpoints: every `60 * 20` ticks (60 s at 20 TPS), storing `{ tick, eventIndex }`. `seek(tick)` walks back to the nearest checkpoint `<= tick`, then filters events from that index.
- `resimulate(matchId)` returns `{ seed, events }` — proves determinism.
- `control(matchId, 'play'|'pause', speed)` — speed allowlist `0.5 | 1 | 2 | 4`, anything else coerced to 1.
- `jumpToEvent(matchId, kind)` returns first matching event's tick.

### `postmatch.ts` — PostMatchService
- Stats shape: `winner`, `finalScore`, plus optional `ratingChange`, `xp`, `mastery`, `missionProgress`, `graphs {population, territory, economy, military}`, `powersUsed`, `energySpent`, `damage`, `healing`, `timeline`, `biggestEvent`, `mvp`.
- `history()` returns match ids in insertion order for the `/history` endpoint.

### `spectator.ts` — SpectatorService
- `join(matchId, kind, ranked)`: ranked → `{ delaySeconds: 90, sensitive: false, visible: { playerId: false } }`; friend/custom/tournament/gm → 0 delay, full visibility. PRD band is 60–120 s; 90 is the midpoint.
- `streamerMode(enabled)` + `view(matchId, frame)`: when on, `playerIds` → `['[hidden]', …]`, `roomCode` → `'[hidden]'`, `popups` → `[]`.

## Routes

| Route | Auth | Notes |
|---|---|---|
| `POST /replay/capture`, `POST /replay/:matchId/event`, `POST /postmatch` | **admin token** | server-authoritative writes; 403 `ADMIN_REQUIRED` |
| `GET /replay/:matchId`, `GET /replay/:matchId/seek?tick=N`, `POST /replay/:matchId/control`, `GET /postmatch/:matchId`, `GET /history` | session | reads/controls for players |
| `POST /spectator/join`, `POST /spectator/streamer` | session | |

Reuse the `adminToken` (`RANKED_ADMIN_TOKEN` env or `options.adminToken`) gate pattern from Phase 7/9 —
same 403 `ADMIN_REQUIRED`, check the token before any session auth.

## Pitfalls hit this round (TypeScript-strict)

- **Class field shadows a same-named method**: `private readonly stats = new Map(...)` + `stats()` method → `post.stats is not a function`. Rename the field (`store`), keep the method name.
- **`noUncheckedIndexedAccess` on fixed arrays**: `players[0]` is `Session | undefined` even after a 4-element literal — use a tuple assertion `as [Session, Session, Session, Session]` then destructure `const [p1, , , p4] = players`.
- **Union narrowing before property access**: `action.powerId` on a discriminated union errors; guard `if (action.type === 'use_power')` first.
- **Literal-param casts for out-of-order tests**: passing an invalid step like `'step-5'` to a typed allowlist needs `as TutorialStep` in the test.
- **`exactOptionalPropertyTypes`**: never assign `undefined` to an optional field; spread conditionally (`...(body.xp !== undefined ? { xp: … } : {})`). Same trap as Phases 7–9.

## Review-round results (fail-closed)

Round 1 passed the 5 acceptance bullets at domain level; failures were TS-strict mechanicals plus
**production-wiring blockers found in Round 1→2 review** — the same blocker class as Phases 7–9
(domain correct, route layer wrong). Reviewer empirically verified each against the built server:

- **Route truncates the domain shape**: `POST /postmatch` mapped only 6 of 17 stat fields — the
  domain type accepted the full shape but the route silently dropped mastery/missionProgress/graphs/
  powersUsed/energySpent/damage/healing/timeline. Fix: map every field in the route and add a
  full-payload round-trip test (assert each field survives GET). A type-level shape is not wire coverage.
- **Declarative privacy is not enforcement**: `spectator.join()` returning `delaySeconds: 90` does
  nothing if `GET /replay/:matchId` still streams seed + events to any session. Fix: capture takes a
  `ranked` flag; replay GET hides `seed`/`events`/`inputs` (`locked: true`) and seek returns
  `423 REPLAY_LOCKED` until a `match_finished` event is recorded.
- **Streamer mode must be per-actor**: a service-level boolean means the last caller toggles hiding
  for everyone. Fix: `streamerMode(actorId, enabled)` keyed by actor; test that another viewer is
  unaffected.
- **Input stream for resimulation**: seed+events alone cannot reproduce a match — player intents are
  input. Fix: `recordInput()` + `POST /replay/:matchId/input` (admin), `resimulate` returns
  `{ seed, events, inputs }`, plus a repeated-resimulate determinism test.
- **Minor**: unknown replay → 404 `MATCH_NOT_FOUND` (was a 500 throw from `resimulate()` — wrap in
  try/catch in the route); event kind runtime-validated → 400 `INVALID_EVENT_KIND`; `matchSeed`
  stored separately from `worldSeed`/`rngSeed` (PRD lists Match Seed distinctly).
- **`exactOptionalPropertyTypes` on `Record<string, unknown>` body spreads**: even conditional
  spreads (`...(body.xp !== undefined ? { xp: body.xp as … } : {})`) fail assignment to a typed
  optional field when the source is `unknown`. Fix: build the object, then `as unknown as PostMatchStats`.

Non-blocking notes: `decide()`-style attack actions may be tier-gated by design; API.md docs lag
behind new route groups (recurring nit — same in Phases 7–9).

## Recurring cross-phase patterns that made this round cheap

- Service-per-domain, injectable via `BuildServerOptions`, instantiated once in `buildServer()`.
- Domain tests first (`replay.test.ts`), then HTTP (`replay-api.test.ts`) with `app.inject()` — no ports.
- Admin-gated writes / session-gated reads is now the house pattern for authoritative endpoints.
- Full closure chain: `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && git diff --check` → fresh fail-closed review → fix blockers → re-review latest tree → announce complete → commit/push only on explicit user request → verify `local == remote` hash after push.
