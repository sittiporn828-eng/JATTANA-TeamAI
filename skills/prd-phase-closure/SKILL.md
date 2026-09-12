---
name: prd-phase-closure
description: "Use when finishing a PRD/TDD phase to acceptance criteria."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [PRD, TDD, phases, acceptance, review, fail-closed, godforge]
---

# PRD Phase Closure (fail-closed)

Finish a PRD-defined phase **completely**, not as a working baseline. User Nat's standard: no rushing, work must be complete per PRD acceptance before any "done" claim.

## Workflow

1. **Read the exact PRD phase section** (scope + acceptance criteria) first. Build an acceptance matrix from it. Never skip this read and never rely on memory of it.
2. **One acceptance at a time, vertical TDD slices**: write failing test → watch it fail (RED) → minimal code (GREEN) → refactor. Repeat. (See `test-driven-development` skill.)
3. **Full gates after every round**: `format:check`, `lint`, `typecheck`, `test`, `build`, `git diff --check`. A green local suite is necessary, never sufficient.
4. **Discover existing context before asking the user for inputs.** Search, in order: current repo docs/PRD/ADR, configured project directories and secondary drives, then session history. Treat a user saying "I already sent this" as a correction: stop asking for the same artifact, show where it was found, and ask only for the smallest genuinely missing contract. Never search an entire system drive indiscriminately; target project roots and text/document extensions, excluding system folders, binaries, node_modules, build output, and caches.
5. **Fresh independent fail-closed review** of the CURRENT tree after EVERY edit round. Dispatch a leaf subagent with toolsets terminal+file, instruct it to inspect files itself ("Do not rely on earlier reviews"), give it the exact PRD acceptance list, and require JSON `{passed, blockers, missing_acceptance, fixes}`.
6. **Do not declare complete, do not commit/push, until the latest review passes** with zero blockers. If the review names blockers, fix them, re-run full gates, and dispatch a NEW review round — never claim prior review covers new edits.
   - If delegation hits rate limits or returns before its JSON verdict, it is not a review result. Inspect the transcript only for leads, then run a local fail-closed matrix; retry a fresh review later rather than treating green gates as approval.
7. For requests covering “every phase”, expand the audit scope to every PRD phase, not only the last changed phase. Track exact acceptance versus production-readiness gaps separately, and report unreviewed phases explicitly.
8. Keep implementation minimal but close concrete acceptance: shared guards at trust boundaries, native browser APIs before dependencies, durable state for operations, and one regression test for each non-trivial slice.
9. Separate exact phase acceptance from broader architecture backlog. If the PRD/DATABASE phase section explicitly scopes a local adapter and names a future production adapter, do not invent the future adapter just to satisfy a broad ADR review; record it as production follow-up, then review the exact phase criteria.
8. After a passing review, only commit/push when the user explicitly asks (Nat: "ไม่ commit/push หากไม่ได้สั่ง").

## Rules

- **Never stop at a hardened baseline when the user asked to close the phase.** Continue through the acceptance matrix, gates, and fresh review; report a baseline only when a real blocker remains and the user did not request completion work.
- **Never announce "almost done" or "complete" from green tests alone.** Green tests prove the slice; acceptance matrix + fresh review prove the phase. Nat explicitly pushed back twice on premature completion signals.
- **A review result is self-reported.** Verify it yourself before trusting: read the delegation summary file, and if needed the live transcript — confirm the subagent actually read files (grep/read_file calls) rather than echoing your context.
- **Review rounds are cheap; wrong completion is expensive.** When in doubt, dispatch another round. The user prefers completeness over speed.
- Run `git status` before/after edits; remove stray untracked artifacts (e.g. downloaded files in the repo root) so they never leak into a commit.
- Note: `phase-based-simulation-development` may overlap with this skill's domain; curator may consolidate.

## Repo-specific shortcuts (Godforge)

- Monorepo: `apps/api` (Fastify REST), `apps/game-server` (WebSocket authoritative, 20 TPS), `packages/simulation`, `packages/shared` (env schema), `packages/network-protocol` (envelope zod).
- Full gate command: `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && git diff --check`.
- Push: `GIT_SSH_COMMAND='ssh -i ~/.ssh/id_ed25519_github_meekamrai -o IdentitiesOnly=yes' git push origin main` — the `work1` key has no rights on `sittiporn828/Godforge`.
- Env for tests: `GAME_SERVER_PORT=0`, `API_HOST=127.0.0.1` for ephemeral game-server tests.

## Pitfalls learned (Phase 5/6/7)

See `references/context-discovery-before-input.md` for the search order, evidence classification, and the Phase 11 provider-context pitfall.

See `references/prd-phase-closure-pitfalls.md` for the full list (auth, allocate tickets, reconnect, event-loop starvation, TS strictness, Fastify route typing, ranked/matchmaking service pitfalls).

## Acceptance granularity and scope freeze

A PRD can be clear at product/phase level while remaining underspecified at implementation level. Before editing, convert each broad criterion into a bounded acceptance row with: required behavior, authoritative producer/state, runtime consumer, automated check, browser/manual evidence, Definition of Done, and explicitly deferred scope. Do not invent extra visual states, evidence requirements, or production-hardening work mid-slice; classify each finding as a blocking implementation gap, required acceptance evidence, documentation gap, or optional enhancement.

Use a scope-freeze rule: once every agreed acceptance row is met, local gates pass, and the tree is clean/pushed as requested, stop. A missing optional screenshot or stronger-than-PRD evidence is not a reason to reopen implementation. Conversely, never promote source inspection/build output into browser evidence. If the PRD does not define a lifecycle, such as settlement collapse progress, record it as deferred instead of implementing a client-only approximation.

For Godforge visual work, distinguish explicitly:
- **PRD acceptance**: the literal phase criterion.
- **Implementation contract**: the minimum state/event and renderer wiring required to make it real.
- **Evidence contract**: the smallest reproducible browser interaction and console check; do not require a screenshot of a state the live product cannot trigger.
- **Deferred enhancement**: stronger polish, extra assets, or broader integration not named by the criterion.

## Playable-product stopping rule

For a large game PRD, define the target release before serial implementation. Use a bounded `Playable Release Candidate` matrix when the user needs something playable: Home → Sandbox → simulation tick → camera → power target/cast → HUD/state sync → accessibility/runtime settings → console health → local gates → clean pushed tree. Split broad PRD work into `P0` playable acceptance and `P1` production-hardening backlog. Once all P0 rows pass, freeze the visual/product scope; do not reopen implementation for optional screenshots, inferred polish, or stronger-than-PRD evidence. A rare state that the current UI cannot trigger is not a blocker if its authoritative implementation and focused test pass; record the unreachable interaction as deferred instead of adding debug-only powers or fake query states. Keep the matrix in a repo document so future sessions use the same stopping point.

## Iterative slice discipline

When the user asks to continue in order, make one vertical work item active at a time. Mark it complete only after its focused test/build passes, then immediately activate the next item. Keep a small todo matrix with the final fresh review as the last item; never mark all implementation items complete while the fail-closed review is still pending.

When a scope-freeze matrix already exists and the user says “continue,” do not reopen P0 or invent visual polish. Select exactly one bounded P1 row, inspect the current implementation and focused tests first, then either close it with existing verified evidence and report no code change, or patch only the concrete lifecycle/trust-boundary gap. For game-service P1 work, prefer existing API/server tests before UI work; a passing queue → allocation → live match → result/release lifecycle is evidence to preserve, not a reason to add another adapter or frontend flow without a stated acceptance row.

### P1 production-hardening slice discipline

For a serial “continue” workflow, inspect the acceptance row and current tree before writing code. Many P1 rows may already be implemented behind service seams; prove them with the narrowest existing focused tests before adding anything. Do not create a duplicate subsystem or tune thresholds merely to manufacture a failing case. If a contract exists but its evidence is missing, add the smallest real regression/rehearsal test against disposable state, then run a fresh independent review of the current tree. Keep implementation, integration-test, and live-runtime evidence separate; a passing service test does not prove browser/deployment behavior.

For backup/restore acceptance, prefer a disposable SQLite rehearsal when the product already uses SQLite persistence: write real wallet/inventory/ledger rows, close the live handle, copy the database file, remove the live file, reopen the copied backup, assert the rows survived, and clean up. This validates a real restore path without inventing a production backup API. Record the ceiling explicitly: file-copy rehearsal is not automatic backup scheduling, PITR, or cloud restore.

For validation pipelines with an HTTP/service boundary, a direct validator unit test is not enough: preserve it, then add one route-level regression proving the boundary status/code/reason (for example, allocation returns 422 with a ranked-rejection code). If the real production state is difficult or unsafe to manufacture, inject only the validator dependency into the existing service-start seam with the production default unchanged; use the injected result only in tests, never tune thresholds or fabricate production data to force a failure. Run focused tests after the seam change and fresh-review the current tree before full gates.

### Godforge visual-slice closure

For Godforge presentation work, prefer the smallest presentation-only slice over new simulation state: inspect the authoritative event/effect payload first, reuse existing Pixi layers and preset/reduced-motion branches, then verify the actual browser interaction before moving on. A slice is closed only with browser evidence, zero console errors, full local gates, a clean pushed tree, and CI status when GitHub access is available. Never claim CI success from a successful push or from an unavailable `gh` query. If a requested visual state lacks an authoritative simulation field (for example construction progress or collapse lifecycle), record it as skipped instead of inventing client-only state. Keep user-requested Ponytail mode: code first, minimum diff, and no feature tour in the completion report.

### Playable RC WebSocket seam pitfall

When hardening a browser client against an authoritative WebSocket, inspect the whole state-ownership path before declaring the seam complete:

- A sparse `entity_delta` containing only `server_tick` and player metadata must not be treated as a complete `WorldState`; either merge a documented delta schema or only apply complete world projections.
- Once an authoritative transport is configured, the local `stepWorld` loop must not continue mutating the displayed state or overwrite server state, including after disconnect/reconnect.
- Validate client configuration at the boundary against the server's canonical civilization/player identifiers before sending `player_ready`; do not rely on the server rejecting malformed configuration.
- Persist and update the session token from `match_found`/`intent_ack`, reconnect with monotonically increasing sequence numbers, and ignore stale server ticks.
- Treat both `world_snapshot` and `entity_delta` as untrusted: require a complete runtime-validated world projection before applying it, and require a non-negative integer `serverTick`/`server_tick` strictly greater than the last accepted tick. Do not advance the stored tick when validation fails.
- A shallow shape guard should cover the fields the renderer dereferences (`map.tiles`, settlements, humans, creatures, resources, active power effects); a TypeScript cast alone is not a trust-boundary check.
- Server integration tests do not prove the changed React client. Classify implementation, server integration, and live browser/backend evidence separately; absence of a real backend browser run remains an evidence gap.

This is the minimum P1 production seam. Do not add fake query-state, synthetic backend evidence, or a new feature to make the test path look complete.

### Playable RC authoritative browser seam

When hardening a browser client against an authoritative WebSocket, inspect the whole state-ownership path before declaring the seam complete:

- Use one shared monotonic client sequence for `player_ready`, `reconnect`, and gameplay intents. Separate counters make the first gameplay intent or reconnect fail as a duplicate.
- Keep configured player and match identity in refs used by action handlers; never hardcode `player-0` in gameplay payloads.
- Validate canonical civilization IDs before sending `player_ready`.
- Disable local `stepWorld` while authoritative transport is configured; otherwise local progression can overwrite server state.
- Treat `world_snapshot` and `entity_delta` as untrusted. Require a complete runtime-validated projection before replacing state, and require an integer, finite, nonnegative, strictly monotonic server tick. Apply the same monotonic guard to snapshots and deltas so stale snapshots cannot regress state.
- Route power/actions through the server intent path when authoritative transport is active. Do not call local `usePower` or mutate client state optimistically without an explicit prediction/reconciliation protocol.
- Store socket/match/player refs used by handlers, retain session tokens for mounted reconnects, and test close → reconnect → snapshot.
- Separate server integration evidence from browser-client evidence. Game-server tests do not prove React frame consumption; report missing live browser/backend evidence instead of inventing it.

### Visual accessibility and evidence pitfall

Every accessibility toggle must be traced to the actual renderer/effect consumer, not merely a root CSS class or checkbox. For Pixi canvas effects, pass the setting into the render function and verify the stage/layer behavior; stale selectors targeting nonexistent DOM nodes are decorative. Reduced motion must suppress both CSS transitions and renderer-side timing/interpolation. Browser acceptance should exercise the real Home → Play → select power → target/cast flow and inspect HUD/state plus console errors; source inspection or synthetic event dispatch is not sufficient. If the browser driver cannot provide visual evidence, report the evidence gap instead of upgrading build/test output into a browser pass. For state-specific visuals, the live path must actually trigger the state: a sandbox that only exposes non-destructive powers cannot prove destruction/rubble rendering. Do not add debug-only powers, query-string fakes, or synthetic state injection solely to manufacture evidence; either use an existing production flow or record the acceptance as unverified.

### Evidence classification for visual slices

Classify each claim separately: `implementation` (source + focused test), `runtime` (live browser state rendered), and `interaction` (real user flow caused the state). A passing implementation/build does not upgrade runtime or interaction evidence. When a destructive/rare state is unreachable from the current UI, leave that acceptance open and name the missing production path.

For production-hardening slices, separate concrete acceptance from stronger architecture claims. A working SQLite-backed atomic limiter, backup rehearsal, or synthetic load smoke is valid evidence for the implemented boundary, but do not relabel it as Redis/global multi-region limiting, PITR, or capacity testing. Record the ceiling explicitly.

After every edit round, rerun the smallest relevant gate before moving on, then rerun the full gate suite after the last slice. If a patch accidentally removes an adjacent declaration/table/JSX block, stop and restore the surrounding context before continuing; migration edits must verify that existing tables such as `op_audits` and `op_risk_flags` remain present.

## Godforge Phase 13–16 closure checks

- A generic global rate limiter or `/metrics` counters do not prove Phase 13/14 acceptance. Test each required route class explicitly (login, redeem, chat, friend request, queue, power, GM login), and inspect bounded cleanup plus multi-instance assumptions.
- GM security is not closed by a bootstrap token and timeout alone: verify MFA, role/permission enforcement, device/IP audit context, and sensitive-action confirmation. Manual risk-flag insertion is not rank-abuse detection; require a production caller that turns suspicious match patterns into a durable flag.
- Allocation acceptance is lifecycle behavior, not an `/allocate` echo. Verify the returned match ID/seed/config are the same state consumed by WebSocket clients, and exercise reserve → running → ending → cleanup/release.
- Native browser audio/notifications and a test button are not Phase 15 integration. Trace the real match-found event to the notification path; check long-string layout, text scale/high contrast, localization-key coverage, and accessible labels.
- A deterministic seed test and a runbook file do not close Phase 16. Require map-balance validation with a ranked rejection decision, integration/security scenarios, and at least one exercised incident/restore path.
- When adding migrations, verify existing tables were preserved (especially `op_audits`) and run migration validation plus restart/persistence tests.
- If delegation returns 429 or max-iteration before emitting the required JSON, classify the review as incomplete; use the transcript only as evidence of inspected files, never as a passing verdict. Re-run a fresh review after every subsequent edit round.
