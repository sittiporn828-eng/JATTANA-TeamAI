---
name: full-stack-flow-closure
description: "Use when verifying every flow across a multi-app system."
version: 1.0.0
platforms: [linux, macos, windows]
---

# Full-Stack Flow Closure

Use when the request is “whole system,” “every flow,” or equivalent and the deliverable must be verified rather than merely audited.

## Scope inventory

1. Enumerate every runnable app/service in the repository before testing: customer UI, API, realtime/game server, workers, admin/GM console, and operational scripts.
2. Build a bounded matrix with rows for:
   - browser navigation and interaction;
   - HTTP routes and authentication/error boundaries;
   - WebSocket join, action, reconnect, disconnect, and release;
   - service-to-service contracts;
   - control-plane allocation/release/result;
   - migrations, maps/data validation, load, release, and operations rehearsals.
3. Treat unit tests as evidence for a row, not a substitute for live cross-component seams.

## Serial closure workflow

For a frozen backlog, close exactly one item through live seam, full gates, fresh review, commit/push, and clean divergence before opening the next. Status reports should distinguish total, completed, in-progress, remaining, and explicitly deferred scope; never silently promote deferred ideas into the active backlog. For token-efficient execution, delegate narrowly scoped implementation to a CLI worker when available, but verify its actual diff and rerun canonical gates yourself; a normal exit or worker summary is not acceptance evidence. Pass the exact absolute repository path in the worker context and require it to `cd` there; if a reviewer inspects another repo, discard that verdict and re-dispatch. Bound worker time and output: poll for progress, kill a worker that is scanning/reasoning without converging, then continue from the surviving diff or use the smallest direct patch. If the worker is unavailable, use the smallest direct patch rather than blocking. Never report completion until independent review, commit/push, and clean-divergence verification are complete.

For Godforge-specific evidence and the validated duplicate/idempotency boundary pattern, see `references/godforge-flow-closure.md`.

1. Confirm baseline branch, working tree, ports, scripts, and environment contract.
2. Run existing tests/rehearsals unchanged first. A stale rehearsal is a product defect: update it to the current trust model rather than weakening production security.
3. Start clean isolated services with explicit non-secret local configuration and unique temporary persistence paths. Verify readiness before probing.
4. Exercise one complete happy path through real boundaries, then fail-closed paths.
5. Browser-test every reachable screen in every UI app, including admin consoles. Check console errors after navigation and significant actions.
6. Test localization, keyboard modifiers/editable fields, accessibility toggles, empty states, and responsive breakpoints when tooling permits.
7. For every reproducible defect, create a tight RED loop, patch the shared root boundary with the smallest diff, and make it GREEN.
8. Restart clean producer and consumer processes after changing built cross-service contracts; stale in-memory state can masquerade as a product bug.
9. Run all rehearsals and full format/lint/typecheck/tests/build/diff-check.
10. Request fresh independent review of the latest diff. Earlier reviews become stale after any production patch.
11. Commit/push only after review passes; verify clean tree and `HEAD == origin/<branch>`.

## Contract checks

At each service boundary, compare the literal wire shape produced and consumed. Do not rely only on injected test stubs: they can share the consumer’s mistaken schema and hide camelCase/snake_case drift. Include a real HTTP/WS seam regression and bind responses to the expected match/job/request ID to reject stale data.

## Rehearsal rules

- Never let a rehearsal regain client authority that production intentionally removed.
- Split responsibilities instead of duplicating giant scenarios: one script owns lifecycle E2E; load smoke may use a small authenticated preflight before load.
- Close sockets/processes in `finally` and bound waits with explicit timeouts.
- Exercise operational fail-closed configuration with local dummy values only; never use or print real credentials.

## Evidence report

Report implemented/runtime/interaction evidence separately. State skipped scope explicitly (for example, persistent notification permission or unavailable viewport resize) and do not label skipped rows as passed.

See `references/godforge-flow-closure.md` for a concrete contract-drift and multi-app example.