---
name: phase-based-simulation-development
description: "Use when implementing a spec-defined simulation/game phase."
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [windows, macos, linux]
metadata:
  hermes:
    tags: [simulation, game-development, deterministic, phase-based-development, tdd]
---

# Phase-Based Simulation Development

Use when implementing a product phase in a deterministic simulation or game from source-of-truth documents.

## Workflow

1. Read the exact phase goal, scope, acceptance criteria, and relevant balance/ADR constraints before coding. Identify rules that prohibit an otherwise convenient implementation.
2. Inspect the existing simulation API and tests. Extend the smallest existing state/update path; do not introduce a parallel simulation model unless the existing model cannot express the phase boundary.
3. Turn every acceptance item into a behavior test. Add negative tests for explicit prohibitions (for example, a war rule must not be a time-only trigger when the balance spec forbids forced wars).
4. Work one vertical slice at a time: RED test, minimal deterministic implementation, focused GREEN test. Keep the seed and all inputs explicit; never use uncontrolled randomness.
5. Model time-based mechanics from events/state transitions, not global clock thresholds, unless the specification explicitly defines a timer. Capture and test the event that starts a hold/cooldown/critical period.
6. Build a requirement matrix with three separate columns: hard acceptance behavior, explicit prohibitions, and quantitative balance/playtest targets. A passing acceptance test does not prove milestone bands such as population, city count, or first-conflict timing.
7. Run milestone probes at every specified checkpoint, not only at the final simulation time. Validate ranges, finite/non-negative state, deterministic replay, and cross-entity ownership consistency.
8. Run the complete quality suite plus deterministic replay probes on multiple seeds before reporting results.
9. Request an independent fail-closed review. Fix every current blocker, then review the latest tree again; a verdict from before concurrent edits is stale.
10. Fingerprint the reviewed source files before verification and again afterward. If any fingerprint changes, re-read the current diff and restart affected checks; waiting for two matching snapshots is a cheap way to avoid reviewing a file mid-write.
11. Treat the last edit as invalidating verification. This includes formatter-only edits: run formatting before the final review, then rerun the affected RED/GREEN check and the full suite after formatting. If execution limits interrupt this closure, report the phase as unverified rather than done.
12. Do not report a phase as complete while the required independent fail-closed review is pending. A green local suite is a verification checkpoint, not the final verdict; fix any review blocker and rerun the full closure sequence.
13. Guard against stale-build false greens in workspaces whose tests import package `main`/`dist`: run a fresh build or typecheck before tests, and fail the review if the build fails even when tests pass against old artifacts.

## State consistency invariants

- Keep competitive starting state symmetric. Use the seed for map generation or deterministic decision tie-breaking, not hidden starting technology, economy, army, or aggression bonuses unless the specification explicitly permits them.
- A conquest transition is atomic across every ownership projection: civilization aggregates, territory/tiles, settlements, buildings, resource nodes, units/population, and event state. Test that the defeated owner no longer appears anywhere it should not.
- Match the public capability surface to the implemented model. If combat state is intentionally single-war/1v1, reject larger match sizes at the boundary; do not accept FFA inputs while global event lookups silently process only one pair. Expand to per-conflict state only when that phase is requested.
- Scope event lookups by conflict and target whenever multiple simultaneous conflicts are supported. Global `find(kind)`/`hasEvent(kind)` checks are only safe behind an enforced single-conflict boundary.

See `references/civilization-war-verification.md` for a reusable milestone and conquest checklist.
See `references/god-powers-competitive-match-checklist.md` for data-driven powers, validation order, energy/cooldown, scoring, and victory closure.
See `references/phase-closure-gates.md` for the concrete full-suite/latest-tree review sequence and recurring false-green failure patterns.
See `references/godforge-phase4-rendering-brief.md` for the verified next-phase handoff from Godforge simulation to client rendering/UX.

## Player-controlled competitive phases

- Start with one end-to-end power tracer: valid God/loadout → server validation → energy spend → cooldown → deterministic world effect. This proves the integration path without pretending the complete power roster exists.
- Keep power definitions and competitive balance numbers in versioned game config. Match state must snapshot the config/balance version; never source costs, cooldowns, score weights, or match duration from environment variables or client-only constants.
- Treat match creation and power intents as trust boundaries: validate unique players/civilizations, God-compatible loadouts, budget, ultimate count, match status, membership, equipped/banned power, energy, cooldown, target, opening protection, and per-player rate limits.
- A timer must not silently replace match state transitions. Test waiting/countdown/running/critical/finished boundaries and reject powers outside `running`. Advance simulation from a positive allowlist of active states (`running`, and explicitly `capital_critical` when intended), not a blacklist of terminal states; otherwise new passive states such as `waiting` or `countdown` can accidentally tick.
- Keep transport/request throttling separate from immutable gameplay transitions unless rejected intents also return updated guard state. A rate limiter stored only in the successful match result cannot count rejected spam and must not be presented as a complete server rate limit.
- Separate a **vertical slice started** report from **phase complete**. Completion requires every acceptance item (all supported powers/effects, score, conquest and time-up victory, rematch state) plus a fresh full-suite run after the final edit.

## Reporting scope honestly

- A passing vertical-slice test proves that slice only. Do not call the whole phase complete until every stated acceptance criterion has a corresponding implemented behavior and passing verification.
- Maintain a per-power behavior matrix, not just a catalog-count test: for each configured power record target selection, authoritative state mutation, timing/expiry, caps, counter interaction, and one boundary assertion. A generic `usePower(...).ok === true` test is dispatch coverage, not effect coverage.
- When concurrent reviewers or coding agents touch the tree, treat every result as scoped to the exact snapshot it inspected. After any edit, rebuild producer packages before tests, rerun the full suite, and request a fresh review of the latest diff; never merge an old PASS/FAIL into the current verdict.
- A catalog count, metadata parse, or `{ ok: true }` dispatch test is not full-roster proof. For every configured power/effect, trace the declared parameters into the authoritative tick/update path and add at least one semantic boundary test (effect onset, magnitude/cap, counter, and expiry where applicable).
- Treat green tests as necessary, not sufficient: perform a fresh fail-closed review against the source-of-truth docs after the last edit. If the reviewer identifies a missing behavior, report **blocked** until that behavior and its test are implemented.
- Label partial delivery as **Phase N baseline/vertical slice** and list the unimplemented acceptance items in one concise line.
- Commit and push only when the user explicitly asks. After a push, verify the remote ref and wait for the commit's CI run before treating the phase as integrated.

## Known closure pitfalls

- Do not let coding agents edit the same working tree concurrently with the host. Their summaries and tests can become stale or overwrite each other; serialize edits, then rebuild and rerun checks from the latest tree.
- A test/debug edit is not complete until temporary `console.log` probes and diagnostic throws are removed and the affected test is rerun.
- For real WebSocket reconnect tests, close/terminate the old socket and wait until server-owned player state is `connected=false` before opening the replacement. Do not call reconnect in the adapter and then through `handleIntent`; mutate authoritative state exactly once, then bind the replacement socket only after acceptance. Use bounded protocol predicates, not fixed sleeps.
- Avoid serializing a full authoritative world to every client every tick. Send a full snapshot on join/reconnect and bounded/projection deltas on cadence or by interest region; otherwise JSON serialization can starve the Node event loop and make reconnect tests time out.
- When a newly added state transition breaks an older phase, inspect the exact event timeline and local state at the transition before changing the rule. Preserve both the new acceptance behavior and the old regression contract; do not weaken a validation gate just to make one test green.
- In pnpm workspaces, tests may import a producer package's stale `dist`; build the producer (or run the root typecheck/build) before targeted tests. A targeted green run before that build is not verification evidence.
- Do not add a second limiter around an existing request guard without an explicit bypass/internal path; otherwise rejected-intent tests can fail because the wrapper and gameplay function count the same request twice. Keep one authoritative counting boundary per transport path.
- Scope in-memory request-attempt and sequence state to the match lifecycle, not only a serialized match ID. Fresh deterministic matches can legitimately reuse the same seed/ID in tests or rematches; a module-global map keyed only by ID leaks prior requests into the new match. Prefer a match-owned guard or a `WeakMap` keyed by the live match object, and add a fresh-same-ID regression test.
- Keep balance values data-driven all the way through: config fields such as growth multipliers, health fractions, damage caps, warning durations, and movement/combat modifiers must be consumed by simulation code rather than replaced by convenient literals or effect metadata that never changes state.
- Derived/buffed state must expire through active effect state. Never permanently mutate `maxHealth`, scores, or other derived aggregates to represent a temporary power unless rollback is explicit and tested.

## World-first presentation handoff (God-game phases)

When converting a deterministic simulation into a WorldBox-inspired sandbox presentation, inspect the actual generated dimensions and landmark coordinates before choosing an initial camera. Do not assume the world is a small square or that the map center contains the capital; in Godforge the world is larger than the viewport and capitals can be at fixed coordinates such as `(24,24)`. Start the camera on a real settlement/capital, then verify zoom/pan against the full map.

Use a visual acceptance ladder: (1) biome separation and terrain composition, (2) settlement/landmark readability at the default zoom, (3) entities and roads visible at medium zoom, (4) HUD subordinate to the world. A palette change or extra procedural decoration does not fix a bad focal point. If browser vision still shows a tile mosaic with settlements unreadable, the phase is not complete even when typecheck, build, and FPS pass.

Keep the render optimization boundary explicit: static terrain/scenery may be cached and redrawn only when camera/viewport/seed changes, while dynamic entities/effects update independently. Prefer deterministic palette variation and existing simulation coordinates over new per-frame objects. Record procedural-rendering limits honestly; WorldBox-like gameplay requires later living entities, authored readability, and direct world interaction rather than claiming parity from pixel colors alone.

See `references/world-first-sandbox-visual-checklist.md` for the reusable inspection and browser acceptance checklist.
See `references/deterministic-authoritative-routes.md` for vehicles/path entities: contiguous cyclic routes, explicit cursors, cargo ownership, cross-field snapshot validation, and multi-seed replay probes.

## Client-rendering handoff (Phase 4)

- Inspect the actual web-game entrypoint and installed dependencies before choosing a renderer. If PixiJS is not already installed and the acceptance gate only requires a first playable projection, native Canvas is the minimal fallback; do not add a renderer dependency speculatively. Record the fallback explicitly so a later Pixi migration is deliberate. If the user asks for the full PRD phase, install/use PixiJS and implement the required product surfaces rather than stopping at the fallback.
- Feed the client from the authoritative simulation package (`createWorld`/`stepWorld` or match state). Do not duplicate world rules in React; the canvas is a projection and HUD derives from the same state.
- Keep the simulation cadence and render cadence separate: use a 60 FPS Pixi ticker with interpolation between the previous and current simulation snapshots; do not claim interpolation when the renderer merely redraws the latest state.
- Full Phase 4 UI acceptance includes App shell, home/lobby, play screen, loadout, in-match HUD/power bar, settings, responsive 1366x768 behavior, keyboard flow, reduce-motion, colorblind baseline, and Thai/English localization. World-render acceptance alone is only a Phase 4 baseline.
- Verify browser behavior, not only TypeScript/build: start the dev server, inspect the rendered Pixi canvas/HUD, navigate Home→Lobby→Play, visit Loadout/Profile/Shop/Settings, change a quality preset, toggle accessibility/language settings, select a power, and verify targeting state changes. Check browser console for JS errors and kill the temporary server afterward.
- Report completion honestly. If any required surface or renderer requirement is missing, label the delivery **Phase 4 baseline/vertical slice**, not complete. The user explicitly corrected premature “Phase complete” claims in this class of task; full-suite green is necessary but does not override missing PRD acceptance items.

## Playable sandbox/client vertical-slice closure

When a phase turns a simulation preview into a playable sandbox, keep the mode boundary explicit: add a Sandbox/Practice entry without silently replacing Ranked behavior. The minimum vertical slice is Home → Sandbox → running deterministic world → pause/resume → speed control → power selection → target click → authoritative simulation command → observable world/energy/cooldown mutation.

For direct world manipulation, keep one coordinate conversion helper and one cast handler. Maintain separate `brushTarget` (hover preview) and locked/last `target` state; never reuse the locked target as the live cursor preview. Route both click and drag-release through the same cast function. Mark a drag as handled and suppress the synthetic click that follows pointer-up, otherwise a drag can cast twice or cast at the wrong endpoint. Clear selected power, preview, and locked target together on Escape. A camera pan must not cast unless the user intentionally dragged with a selected brush; verify the movement threshold at the input boundary.

A selected button, target ring, toast, or clean browser console is not proof that gameplay works. Trace every interaction through the domain command and assert a state change in the browser (for example energy `40 → 34`, population/tile/entity change, or cooldown activation). If the UI only changes `selectedPower`, it is a preview, not a playable feature. Keep Sandbox controls localized in EN/TH and verify Pause actually suppresses the simulation step callback, while speed changes cadence without changing deterministic outcomes.

A selected button, target ring, toast, or clean browser console is not proof that gameplay works. Trace every interaction through the domain command and assert a state change in the browser (for example energy `40 → 34`, population/tile/entity change, or cooldown activation). If the UI only changes `selectedPower`, it is a preview, not a playable feature. Keep Sandbox controls localized in EN/TH and verify Pause actually suppresses the simulation step callback, while speed changes cadence without changing deterministic outcomes.

Report the phase as a baseline/vertical slice when only a subset of the power roster is equipped or when the world presentation is still procedural; do not claim WorldBox-level gameplay from a working canvas alone. Record the exact playable subset and leave Ranked/online authority for its own phase.

## Online server/networking closure (Phase 5)

When a phase adds a WebSocket game server, treat transport as a trust boundary, not a relay:

- Bind `socket -> playerId` on the server. Never prefer a client-supplied `player_id` on an already-bound connection; reconnect on a new socket must explicitly authenticate/rebind before accepting intents.
- Route `use_power` through the authoritative simulation API (`usePower`/equivalent) before broadcasting. A `power_event` is an output of accepted server state, never a broadcast of untrusted payload. Whitelist event fields so client payload cannot overwrite server identity.
- Keep the server core importable and side-effect free for tests. Put `listen()`/timers in a separate process entrypoint (`main.ts`); importing the server module must not open a port or start a 20 TPS loop.
- Test at least two real WebSocket clients: same match ID, same authoritative snapshot/delta, power event visible to the other client, forged identity rejected, duplicate sequence rejected, disconnect/reconnect without a duplicate player.
- Make integration tests wait for the expected protocol message/predicate with a bounded timeout; fixed sleeps create parallel-suite flakes and do not prove the event arrived.
- Include authoritative world/match state in snapshots/deltas, not only player metadata. Build/typecheck the simulation producer before server tests to avoid stale workspace `dist` false greens.
- Fresh review is mandatory after the last edit. A green suite does not close missing authentication, persistence, message-size/freshness limits, match lifecycle, or API matchmaking acceptance items; report the phase as baseline/vertical slice until those are implemented.
- For fail-closed Phase 5 review, trace the real socket flow and verify the auth boundary before trusting tests: first join must use server/API-issued identity or a match ticket (never raw client `player_id`), the join acknowledgement must actually deliver the reconnect credential, and reconnect must be tested end-to-end on a new socket. Also check explicit WebSocket `maxPayload`, timestamp age/future bounds, transport-level connection/message throttles, deterministic seed provenance, duplicate-socket replacement/close races, camera-interest behavior, and required lifecycle messages. Treat a power broadcast built from accepted client fields as insufficient until the test proves authoritative energy/cooldown/effect/world state reaches the peer.
- Scope review against the exact phase acceptance criteria before importing requirements from later phases. For Godforge Phase 5, the hard acceptance is seven online-match behaviors: two players in one match, shared server truth, cross-client power visibility, duplicate suppression, correct reconnect state, no duplicate on disconnect/reconnect, and continued simulation while disconnected. Phase 6 account identity and Phase 7 matchmaking are explicit follow-ups unless Phase 5 itself requires that boundary. Do not use a green vertical slice to claim completion, but do not block on later-phase features without labeling the scope decision.

## Allocation control-plane trust boundary

For authoritative game-server allocation endpoints, authenticate before parsing or mutating lifecycle state. Require a dedicated server-to-server token (for example `x-game-server-token` matched against `GAME_SERVER_ALLOCATE_TOKEN`) and fail closed when the token is absent or wrong; the API caller must forward the same configured token. Validate the allocation match identifier against the protocol's actual envelope contract (UUID v4 in this stack) before setting `activeMatchId` or entering `reserved/running`. Add one runtime probe/test for missing token, wrong token, invalid UUID, and valid UUID. A syntactically non-empty ID is insufficient because it can poison outbound envelopes after lifecycle mutation.

After fixing a review blocker, rebuild producer packages before targeted tests, rerun the full gates, and dispatch a fresh review of the current tree. Never reuse a pre-fix PASS or infer trust-boundary closure from unit tests that bypass the HTTP/WebSocket boundary.

## Social/API layer closure (Phase 6)

When a phase adds accounts, friends, party, or custom rooms to a Fastify API, keep the domain service injectable and test through `app.inject()` (no port needed). Fastify 5 specifics, actor-semantics test traps, and the post-review hardening pass (session-token auth, scrypt, JSON persistence, real `/allocate`) are in `references/godforge-phase6-social-api.md` — read it before writing routes or tests for this class of phase.

## Ranked/multi-mode closure (Phase 7/8)

When a phase adds matchmaking/rank (Elo, placement, season, remake) or team modes (2v2, FFA, pick/ban), the recurring blocker class is **production wiring and route authorization, not domain math** — green unit/domain tests routinely miss it. Before declaring complete, grep callers of every domain function (a tick/expansion loop with a test but no caller is dead in production), check busy/state cleanup on every early-return path, and require match-membership authz on result/remake-style routes plus an admin gate on global-control endpoints. Elo/placement/season semantics, the Phase 7 blocker list with fixes, TeamSystem/FFA/DraftRules shapes, and TypeScript-strict pitfalls (`exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, Fastify 5 params typing) are in `references/godforge-phase7-8-ranked-modes.md` — read it before reviewing or extending this class of phase.

Multi-player modes (2v2/FFA) add their own blocker class — see the Phase 8 round in the same reference: a 1v1-shaped result flow strands the extra participants busy forever (result must carry every participant id and clear busy for all of them), domain helpers without route callers are dead code, test fixtures must never be baked into production state (server must re-derive FFA elimination from match state, not trust a client-filtered `active` list), matchmaking responses need the full roster so team assignment is drivable, and global mutable maps keyed by nothing (e.g. a module-level team-assignment map) must be removed in favor of pure functions.

## Progression/bot closure (Phase 9)

When a phase adds tutorial, bots, XP/level unlocks, per-god mastery, missions, or achievements, keep each domain service injectable and test domain-first then HTTP (`app.inject()`). Tutorial steps must be an ordered allowlist (reject out-of-order with 409), bot difficulty is a static-method service with an `isBot` guard (never into ranked), and missions reset per cadence (`resetDaily` must not touch weekly). Watch for self-awarded progression exploits: a raw `POST /progression/xp {amount}` route is a rank-unlock/grind vector — prefer server-awarded XP from authoritative events. Shapes, routes, TypeScript-strict pitfalls (union narrowing before property access, literal-param casts, `typeof` for static-object services), the PRD `python find()` reading quirk, and delegation/review re-dispatch quirks are in `references/godforge-phase9-progression-bots.md`.

## Replay/post-match/spectator closure (Phase 10)

When a phase adds deterministic replay, post-match stats, spectator feeds, or match history, keep each domain service injectable and test domain-first then HTTP (`app.inject()`). The replay contract is **seed bundle + input/event stream**: capture world/RNG/simulation/balance/map versions plus an initial snapshot, then resimulation is a pure replay of that stream. Checkpoints belong at a fixed tick cadence (60 s at 20 TPS = 1200 ticks) so `seek` starts from the nearest checkpoint instead of scanning from tick 0; replay controls are an explicit speed allowlist (0.5/1/2/4). Post-match stats and replay event ingestion are **server-authoritative writes — gate them behind the admin token**, never a client POST; reads (replay/seek/history/stats) take a normal session. Ranked spectator feeds must delay 60–120 s and redact player identity; streamer mode hides player ids, room codes, popups, and friend requests. Blockers to expect: a class field that shadows a same-named method (`stats` Map vs `stats()` — rename the field), `noUncheckedIndexedAccess` on fixed-size arrays (use tuple assertions), union narrowing before property access on discriminated action types, and literal casts when testing out-of-order input against a typed allowlist. Round-2 blocker class (wiring, not math): the route must map **every** stat field the domain type declares (round-trip test it — a type-level shape is not wire coverage); a spectator delay contract is not enforcement — lock replay reads (`locked`, `423 REPLAY_LOCKED`) until `match_finished` for ranked matches; streamer mode is per-actor, not a service singleton; capture player intents (`recordInput`) so seed+inputs actually resimulate. Full shapes, routes, and the review-round results are in `references/godforge-phase10-replay-postmatch.md` — read it before reviewing or extending this class of phase.

## Store/economy closure (Phase 11)

When a phase adds a store, currency, inventory, redeem codes, bundles, season passes, or payments, use one injectable economy service as the authoritative boundary.

- Wallet, ledger, ownership, purchases, redeems, and provider receipts must be durable and transactionally committed together. Use normalized, migration-managed database tables—not a process-local map or a single serialized JSON snapshot row. Require unique constraints for provider transaction IDs and `(redeem_code, player)`, plus an atomic limited-code reservation, so separate API instances cannot double-grant or overwrite each other's state.
- Derive region, level, rank, and season from server-owned account/ranked state; never trust client region or eligibility fields. Validate every product/redeem/payment input at the HTTP boundary.
- Product definitions must *enforce* every data field: category, region/window, active state, account limit, discount, and reward. Bundles need a deterministic partial-ownership rule; test it. Model and test both free and premium season-pass tracks with cosmetic-only rewards.
- Wallet mutation appends a ledger entry in the same transaction; inventory ownership stays server-side. Cosmetics must have no path into ranked calculation.
- Never accept client `payment_success`. Verify a signed provider event plus its authoritative transaction/product/player/amount/status server-side, write a receipt, then grant once. An injected verifier must default fail-closed.
- Test domain idempotency and `app.inject()` routes separately, including duplicate callbacks, region spoofing, every redeem predicate, malformed input, partial bundles, restart, and multi-instance behavior.

### Phase 11 review lessons

- A durable file-backed SQLite adapter plus runtime DDL is a useful vertical slice, but it is not equivalent to a PRD that names PostgreSQL and migration-managed production schema. Keep the migration file, copy it into build artifacts, and report the adapter as a baseline until the production database adapter is implemented.
- Treat every commerce route as a trust boundary: reject malformed reward/config fields rather than silently dropping them; require the authenticated account to exist; derive region/rank/season from server-owned services; and reject client payment-success claims before any economy call.
- Tests must cover restart persistence for wallet, inventory, ledger, purchase/redeem state, and receipts—not only two sequential service instances. Add semantic tests for free/premium season-pass claims and malformed redeem definitions.
- An injected `paymentVerifier: () => true` proves wiring/idempotency only. It does not prove provider signature, amount, product, player, or status verification; keep provider-specific verification as an explicit production blocker until an actual adapter is supplied.
- After modifying a phase, rerun formatting before fresh review, then full typecheck/build/test/lint/diff gates and a new fail-closed review of the latest tree. Never merge a pre-edit review into the post-edit verdict.

See `references/godforge-phase11-economy.md`. 

### Phase 11 review lessons

- A durable file-backed SQLite adapter plus runtime DDL is a useful vertical slice, but it is not equivalent to a PRD that names PostgreSQL and migration-managed production schema. Keep the migration file, copy it into build artifacts, and report the adapter as a baseline until the production database adapter is implemented.
- Treat every commerce route as a trust boundary: reject malformed reward/config fields rather than silently dropping them; require the authenticated account to exist; derive region/rank/season from server-owned services; and reject client payment-success claims before any economy call.
- Tests must cover restart persistence for wallet, inventory, ledger, purchase/redeem state, and receipts—not only two sequential service instances. Add semantic tests for free/premium season-pass claims and malformed redeem definitions.
- An injected `paymentVerifier: () => true` proves wiring/idempotency only. It does not prove provider signature, amount, product, player, or status verification; keep provider-specific verification as an explicit production blocker until an actual adapter is supplied.
- After modifying a phase, rerun formatting before fresh review, then full typecheck/build/test/lint/diff gates and a new fail-closed review of the latest tree. Never merge a pre-edit review into the post-edit verdict.

## GM console/LiveOps closure (Phase 12)

When a phase adds a GM web console, support tickets, bug reports, moderation, liveops, or admin read models, the blocker class is **real-data wiring and durable ops state, not CRUD** — a `matches: []` stub, a process-local ops store, decorative feature flags, and a shared static admin token are the recurring review blockers. Fixes: session-token login (`/admin/login` + expiry) instead of a static token on admin routes; a live-match registry recorded in `startMatch` after `allocate()` succeeds and closed by mode+participant overlap on result/remake; ops persistence as migration-managed SQLite (mirror the economy service's `node:sqlite` + `002_operations.sql` pattern, keep the service API identical); feature flags consumed by real routes (`isFeatureEnabled`, default true); a public `audit()` called from every sensitive admin route (redeem create, economy credit, flags). When resuming mid-phase work, read the newest `~/AppData/Local/hermes/cache/delegation/subagent-summary-*.txt` first — it is the exact blocker list with evidence/line refs. Shapes, test patterns (restart persistence, route-level kill switch, live monitor via injected `allocate`), and verification order are in `references/godforge-phase12-gm-console.md`.

## Execution-status integrity

- Never say work is continuing, promise a completion notification, or quote progress while idle. A chat turn does not keep coding running after the response. For a long phase, either keep executing tools in the current turn or start a real tracked background process and report its verified handle/status; if that process fails, state the failure immediately and do not imply code changed.
- Do not estimate a completion time before the implementation path is validated. Report verified milestones instead: files changed, focused test result, full gates, then the fresh review verdict.
- Do not create a one-shot cron job as a substitute for an active coding loop. Cron model capacity, isolated context, or dispatch failures can leave the user waiting without repository progress. Use it only for genuinely deferred work and say it is best-effort; for an active phase, keep executing in the current turn.
- When a user asks to wait for completion, keep the phase marked in progress and only announce completion after the final review and requested commit/push verification.

## Client observe/inspect vertical slices

For an inspect/observe mode in a living sandbox, keep selection presentation-only and separate from authoritative mutation:

- Add a pure helper that accepts the current `WorldState` plus tile position and returns a compact discriminated inspection result or `null`; prefer nearest living human within the requested radius, then active settlement within its requested radius.
- Derive population/stage/health from the authoritative world projections already present. Do not infer settlement population from a partial rendered/filtered entity list when civilization population exists.
- Keep Inspect mode orthogonal to cast mode: canvas clicks branch to inspection only while the mode is active; cast handler and selected power behavior remain unchanged otherwise. Clear inspection/hover/target presentation state when toggling modes, and expose the icon-only action with an accessible label and pressed state.
- Test the pure helper first with living/dead, radius, nearest-target, settlement fallback, authoritative population, and no-result cases. Run the package-scoped focused test RED before adding the helper, then GREEN; a missing module is acceptable RED, but fixture/type errors are not.
- Preserve unrelated uncommitted UI work. Inspect `git diff` before and after formatting; format only touched files and verify `git diff --check`, because broad formatter runs can rewrite pre-existing dock/style changes.

## Ponytail constraints

Prefer one state model, one step function, and existing test tooling. Do not add generic AI frameworks, factories, configuration layers, or renderer/network code for a simulation-only phase. Never simplify away source-of-truth fairness, security, determinism, or explicit gameplay constraints.
