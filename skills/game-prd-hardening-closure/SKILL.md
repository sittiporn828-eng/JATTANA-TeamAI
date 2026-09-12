---
name: game-prd-hardening-closure
description: "Use when closing a game PRD after hardening edits."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [game, PRD, acceptance, hardening, security, recovery, release]
---

# Game PRD Hardening Closure

Use for an authoritative multiplayer game or simulation whose late phases combine security, operations, client accessibility, deterministic simulation, and release readiness. This is a closure workflow, not a green-test checklist.

## Fail-closed workflow

1. Read the exact PRD phase sections and build a criterion → code path → regression test matrix.
2. Split remaining blockers into small ordered work items. Do exactly one item at a time; do not start the next item until the current item has a focused regression test and its build/tests pass. Keep a visible task list with only one item `in_progress`.
3. Inspect the real path end to end: API input, durable store, allocation/service boundary, WebSocket state, client event, and release/operator path.
4. For every claimed fix, identify a test that would fail if the behavior regressed. A helper, CLI, settings toggle, or endpoint that is never called in production is decorative.
5. Run the focused gate after each work item, then run the full gates after the final edit: format, lint, typecheck, tests with reconciled counts, build, migration validation, and `git diff --check`.
6. Dispatch a fresh read-only reviewer against the current tree after every edit round. A prior review does not cover later edits; rate-limited or truncated reviews are incomplete.
7. Do not commit/push or claim closure until the latest review has zero blocking findings.
7a. A reviewer timeout or truncated result is not a pass and is not complete evidence. Continue with bounded local checks or dispatch a fresh review; never claim `passed:true`, commit, or push from timeout/truncated output.
8. When a fresh reviewer returns blockers, treat the result as the next work queue: fix only the first blocker, add its failing-if-regressed test, run the focused gate, then continue. Do not answer with a backlog-only status update.
8a. When a user is waiting on serial work, report only verified item completion and immediately continue the next item; do not let a long-running aggregate review hide concrete progress.
9. If the repository has separate durable stores (for example economy and operations SQLite files), rehearse backup and restore for each profile independently; verify integrity, required tables, and at least one restored row count.
10. Recovery acceptance requires all three links: a producer that creates the recoverable state, an idempotent transactional recovery method, and a production trigger (startup loop, scheduled worker, or webhook path) plus a regression test.
11. Operational metrics acceptance needs an executable exposure and check: labeled live metrics, a dashboard/summary endpoint, and alert evaluation/rehearsal. Counters alone are not a dashboard; a green unit test alone is not alert delivery evidence.

## Required late-phase checks

- Security: explicit route policies for login, redeem, chat, friend request, queue, power, and GM login; bounded cleanup; MFA/RBAC/confirmation/device audit where required; automated suspicious-pattern detection must create durable risk flags.
- Allocation: API-issued match ID, seed, and config must become authoritative server state consumed by WebSocket validation and envelopes. Exercise duplicate allocation, lifecycle reserve/running/release, and reconnect.
- Payments: pending records must be durable; recovery must be transactional and idempotent, use the same ledger/uniqueness path as original grants, and have a production caller plus regression test.
- Observability/recovery: metrics must be live, lifecycle counters must reflect real transitions, backup/restore and incident paths must be executable rather than prose only.
- Client UX: notification/audio settings must trace from the real match-found event to native APIs with listener cleanup. Check localization-key coverage, long strings, text scale, contrast, and accessible labels.
- Determinism/release: seeded checkpoint tests are necessary but insufficient. Map validation must fail closed with measurable reasons and be wired into ranked admission or an explicit release gate. Add integration/security scenarios and rollback/runbook evidence.

## Evidence rules

- Generic `/metrics`, a runbook, a validator CLI, or an HTTP response echo is only a slice of acceptance, never proof of integration.
- Never convert a green full gate into a full-acceptance claim. Keep separate statuses for implementation gates and the latest fail-closed PRD review; unresolved blocking findings remain active work even when all tests/builds pass.
- When the user asks for serial execution, finish and verify exactly the first blocker before starting the next. After each focused fix, report the concrete evidence briefly and continue; do not stop at a backlog-only summary.
- After a targeted text patch to JSX/TSX or nested markup, re-read the edited region before formatting/building. Replacement patches can silently remove a parent tag or sibling declaration; the cheapest detection is a fresh local slice plus typecheck.
- If a delegated reviewer returns a truncated summary, read its saved full artifact before acting. Treat only the complete structured verdict as review evidence; do not infer omitted blockers.

- Preserve existing migration tables when adding schema; after every migration edit, verify that all prior tables still exist and validate restart persistence and audit rows.
- Trust-boundary numeric inputs must reject non-finite, fractional, negative, and coercion-poisoned values before domain mutation; add a route regression test for `NaN`/`Infinity`/fractional input, not only a happy-path type assertion.
- Every custom-room/server allocation needs a production terminal release seam, not just ranked result/remake cleanup: authenticate the room host, call the real game-server release using the stored authoritative server URL/match ID, then clear the room binding only after the release path is invoked. Test allocation failure, release authorization, and retryability.
- For a rate limiter, prefer an atomic durable counter (`BEGIN IMMEDIATE`/equivalent) over a process-local map when multiple API workers share the same SQLite store. Explicitly document the ceiling: shared-file/node safety is not cross-host distributed safety.
- Allocation is not complete until the API stores the returned server URL **and returned authoritative match ID** per allocation, the authoritative server consumes the requested match ID and seed, and result/remake/abort paths call release using the stored ID—not a reconstructed `playerIds.join('-')`. Inject the release seam in tests and assert a deliberately different allocator ID reaches the release call; also test that failed release retains state for retry and successful release clears it.
- A real API/game-server rehearsal must start the executable service entrypoint (`main`/supervisor entry), not a library module that only exports `startGameServer()`. Drive guest/session creation, queue, allocation, result, and release over HTTP against two actual processes; run load scenarios before E2E or restart services because in-memory matchmaking state can intentionally consume a match.
- Browser match-found acceptance requires a real WebSocket handshake: deployment-provided URL, match ID, player ID, and session context; send the protocol's `player_ready` envelope on open, then trigger notification from the server `match_found` envelope. A local synthetic event button is only a test adapter, never integration evidence.
- Determinism acceptance should hash a locked match contract (match ID, seed, config version, map, ordered input records) together with canonical checkpoints and replay it twice. Primitive `createWorld/stepWorld` equality is useful groundwork but is not sufficient evidence for a competitive match contract.
- Load acceptance should have an opt-in authenticated scenario (guest creation → queue → matchmaking) in addition to health/metrics probes. Keep it isolated from E2E or make it clean up/restart services so one intentionally live match cannot create a false negative in the next rehearsal.
- Runtime SQLite/temp artifacts must be ignored by repository patterns (`data/`, temp backup directories) rather than deleted blindly during cleanup; inspect contents first and preserve user data unless explicit deletion consent exists.
- Alert acceptance needs both evaluation and a delivery path. A dashboard that only returns alert names is insufficient; provide an injected test sink and a configured webhook/dispatcher path, and exercise the sink with a forced alert in a focused test.
- Release correctness has two independent checks: the API release caller must reject non-2xx responses before deleting its allocation binding, and the game-server must close/clear every socket, player/session, active-connection, sequence, and authoritative-state map before returning `available`. Test both sides separately.
- Metrics scope must cover the acceptance vocabulary, not only generic HTTP counters: expose live matchmaking, payment-recovery, rate-limit, trace, server tick/connection/lifecycle, and release-failure signals. If a counter cannot be incremented by a production caller, it is decorative.
- Load acceptance should make the authenticated scenario the default rehearsal path, not an opt-in flag: create guests, queue both players, allocate, submit a result, verify release, then run concurrent probes. If a scenario intentionally leaves in-memory state, clean it up or restart services before the next rehearsal.
- Settings acceptance is behavioral: a stored audio/accessibility value that is not consumed by a real render/audio path is decorative. Either wire it to the native bus/render effect or remove the control; do not count a slider as coverage.
- Release rehearsal should be one executable command that chains endpoint readiness, authenticated match lifecycle, authoritative release, and any required restore/incident checks. Separate scripts are useful primitives, but the package-level rehearsal must compose them.
- When a package script chains commands with `&&`, positional arguments may reach only the final command; use explicit environment inputs such as `API_BASE_URL`/`GAME_SERVER_HTTP_URL` for every chained stage, and test the exact package command rather than each script separately.
- Live verification must rebuild and restart the executable service after source changes; an already-running `dist` process can make a correct metrics/release fix appear absent. Record the executable entrypoint (`dist/main.js` vs library module) in the rehearsal command.
- For serial user-directed work, close each item with its focused test/build evidence before opening the next, and report the item as done immediately; do not wait on a later aggregate review to communicate completed work.

- A recovery script or backup CLI is not evidence until it is exercised against a temporary database and the restored database is opened, integrity-checked, and queried. If rehearsal fails, report it as unverified; do not promote the attempted command into a workflow.
- Distinguish exact PRD acceptance from broader production architecture gaps such as cross-host rate limiting or external tracing.
- RBAC must be enforced by the stored session role at the sensitive route, not merely by a valid admin session. Keep `operator` read/low-risk access separate from `admin`/`superadmin` mutation access; make the configured role mapping explicit and test an operator session receiving 403 on a sensitive mutation.
- Durable rate limiting also needs bounded retention. A `BEGIN IMMEDIATE` counter without periodic deletion still permits attacker-controlled key growth; use an atomic, amortized cleanup of expired rows with a documented retention window and a focused persistence test.
- Localization acceptance is a source-coverage check, not just a locale selector. Scan all rendered user-facing JSX strings, including HUD labels, settings controls, status text, tutorial/power/shop/error copy, and footer/help text; every rendered string needs a key in every supported locale. Build/typecheck only proves dictionary shape, not coverage.
- Audio settings acceptance requires independent, audible sources on each claimed bus. Connecting empty gain nodes or routing one notification oscillator through multiple controls is decorative; either connect real music/ambient/power/UI/notification sources and test channel isolation, or narrow/remove the controls.
- Map acceptance must enforce every reported balance dimension. A validator that reports `expansionSpeed`/`firstContactTick` but never applies thresholds is not fail-closed; add explicit ranked thresholds, rejection reasons, and custom-mode allowance tests.
- If a simplified local implementation is deliberate, mark its ceiling and upgrade path instead of implying horizontal production readiness.

## References

See `references/late-phase-game-review.md` for the reusable matrix and evidence checklist.
See `references/rehearsal-patterns.md` for the validated two-process E2E, release-binding, browser-handshake, and artifact-cleanup patterns.
See `references/operational-rehearsal.md` for the validated backup/restore, composed release rehearsal, kill-switch, and signed payment rehearsal commands.
See `references/final-closure-techniques.md` for the executable alert probe, real input replay, and localization source-coverage patterns.
