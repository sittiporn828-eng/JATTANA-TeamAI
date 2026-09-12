# Godforge Phases 13–16 Review Notes

Use this as a session-specific checklist when reviewing the Godforge master PRD against the current tree. It is evidence guidance, not acceptance by itself.

## Fresh-gate baseline

Run independently from the repo root:

```text
pnpm validate:maps
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm validate:migrations
git diff --check
```

Record actual test counts from the output. A passing gate proves build/test health only; it does not prove production integration.

## Allocation/release seam

Trace both sides of the lifecycle: API match start -> allocation -> running -> match completion -> release -> server available. Verify the API actually invokes release; a game-server `/release` endpoint alone is insufficient. Probe a real server and inspect authoritative state before and after release. Require the post-release match id, world, players, sequence/session maps, and connections to be reset consistently. Also verify release is authenticated/authorized or bound to the active allocation; an unconditional release endpoint is not a safe lifecycle.

## Validator integration

A CLI validator is only a pipeline check. Read the ranked allocation path and confirm it calls the validator before accepting ranked matches. Map acceptance must compute the PRD metrics (side win rate, resource access, expansion speed, first contact, population growth, territory distribution) from a deterministic bot simulation or equivalent evidence. Structural checks such as population/territory/capital distance are partial only.

## Security and abuse

Distinguish durable storage of a manually submitted risk flag from automated abuse detection. Search production callers for detector logic. For result/report routes, prove the authenticated actor is a participant/admin authorized for that exact match; checking that arbitrary IDs are merely "busy" is not authorization. Optional MFA, direct bootstrap-token bypasses, and stored-but-unused roles are fail-closed blockers for a required MFA/RBAC acceptance.

## Payments/recovery

Trace the whole transaction and failure path: provider verification -> pending state -> retry/recovery -> single grant -> receipt/ledger. A `recoverPendingPayments()` helper without a production pending-state producer, retry worker, or restore test is not payment recovery acceptance. Check owner-only receipt access and provider SKU/amount/currency binding separately.

## Observability/release operations

Minimal counters and request logs are not the PRD's metrics/traces/alerts/dashboard. Check required correlation fields (`request_id`, `trace_id`, `player_id`, `match_id`, `server_id`, `version`), simulation/tick metrics, DB/error/disconnect health, alert rules and delivery, and actual dashboard exposure. Runbook prose is not a rehearsal: require executable backup/restore, kill-switch, incident, and end-to-end evidence.

## Localization/accessibility/notification wiring

A locale selector with a few conditional strings is partial unless the UI consistently consumes keys and long strings are exercised. A reduce-motion CSS class must affect the actual expensive/rendered effects, not only transitions while a simulation/ticker continues unchanged. Browser notification permission plus a local test event is not match-found wiring; trace the real backend/WebSocket event into the browser. Check the full required control set: scale/text size, colorblind, high contrast, reduce motion, screen shake, subtitles, and separate audio controls.

## Evidence discipline

For each claimed fix, read current source and trace production callers, then run a smallest real probe at the seam. Tests that only assert response shape or a mocked allocator do not prove lifecycle correctness. Keep final output strict JSON when the parent task specifies a schema; include `passed`, blocker objects with severity/finding/evidence, `missing_acceptance`, and a per-criterion matrix. Do not modify the reviewed tree.

## Fresh-review false positives to reject

- **Determinism contract is not decorative input metadata.** If a test declares recorded inputs, verify those inputs are actually applied at their recorded ticks through the competitive-match path before hashing checkpoints. Hashing `{contract, tick, world}` while only calling `createWorld()`/`stepWorld()` proves primitive replay, not match-contract determinism.
- **Metrics endpoint is not alerting.** A `/metrics` or `/metrics/dashboard` JSON response with counters and a few derived conditions is not proof of the PRD's critical alerts. Map each required alert to a producer, trigger, delivery path, and executable rehearsal; polling a dashboard endpoint alone is insufficient.
- **Localization must be exhaustive for visible UI.** After dictionary wiring, scan rendered JSX for remaining user-facing literals, including labels, control states, app chrome, notification text, and raw domain identifiers. One hardcoded visible string means runtime locale coverage is partial.
