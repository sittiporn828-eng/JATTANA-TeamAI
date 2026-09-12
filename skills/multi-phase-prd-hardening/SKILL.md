---
name: multi-phase-prd-hardening
description: "Use when closing multiple PRD phases with one final commit."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [PRD, phases, acceptance, hardening, live-seams, release]
---

# Multi-phase PRD hardening

Use when a user asks to continue through several remaining PRD phases and commit/push only once at the end. This complements phase-specific acceptance reviews; it does not replace them.

## Workflow

1. Read the exact acceptance section for every target phase and create a phase matrix: criterion, producer, consumer, durable state, regression, focused gate, fresh-review status.
2. Inspect the current tree and git state before editing. Keep one concrete blocker in progress at a time, but retain the whole matrix so later phases are not silently skipped.
3. For every cross-service criterion, trace a real non-test producer and consumer. A unit-tested service method, admin-only injection route, or in-memory Map read is not production wiring.
4. Implement the smallest real vertical slice and add a regression that fails if the behavior regresses. Run the focused gate before moving to the next blocker.
5. After each edit round, dispatch a fresh read-only review against the current tree. A previous review does not cover later edits. Truncated or timeout results are incomplete, not passes.
6. Run the full gates only after the final implementation round, then run one final fresh review. Commit and push once—and only once—the final review has zero blockers and every phase in the matrix is reviewed.
7. Consume async review completions immediately: a completed review is authoritative current state, not a pending status. Apply its blockers or explicitly stop; never continue from an earlier optimistic summary.
8. Do not turn a green test suite into an acceptance claim. Keep reporting implementation progress separate from phase closure; unresolved live-seam findings remain blockers until a fresh review verifies them.
9. Never satisfy missing authoritative telemetry with constants, zero-filled fields, fabricated lifecycle flags, or admin/test-only producers. Either trace the real producer through the terminal seam or leave the criterion incomplete.
10. If the user requests one final commit after many phases, preserve a clean audit trail: no intermediate commit/push, but still run focused gates after each risky edit and full gates plus final review immediately before the single commit.
11. If the run cannot reach that state, report the exact verified stopping point and leave the tree uncommitted; never promise that background work completed.
12. Treat a fresh review as invalidated by any subsequent tree change. Re-run focused checks, dispatch a new review, then rerun full gates before the final review; never reuse a prior verdict after edits.
13. When an async review completion arrives, consume it immediately. If it finds a blocker, patch the current source and tests; do not merely report that review is pending or summarize an optimistic earlier pass.
14. Never use zero-filled metrics, hardcoded rating/XP, fabricated release flags, or client-supplied frames as substitutes for authoritative producers. If the real producer cannot be wired, leave the phase open and report the exact gap.
15. For replay/spectator/e2e work, prove both sides of every seam with a live or executable probe: game-server input/event → replay ingest → reducer/result; allocation → authenticated release → available lifecycle; authoritative frame → redaction/streamer consumer. Health-only probes and admin-only injection routes are insufficient.
16. Validate every rehearsal script with `node --check` before treating it as evidence. For asynchronous lifecycle transitions, poll with a bounded timeout until the authoritative state changes; an immediate second metrics read is not proof.
17. When an async review completes, read the complete result (not only the truncated summary), patch blockers immediately, and tell the user progress only after doing work. Never say you are merely waiting while a completed review or actionable tool result is available.
18. Any tree edit invalidates the last review and full-gate claim. After the final patch: focused regression → fresh review → full gates → final status/diff check → single commit/push. No exceptions.
19. Before declaring the playable artifact ready, run a real browser smoke path against the current build: start the web app with the actual logged URL, navigate Home → Sandbox/Play → Settings, exercise one interaction, inspect accessibility controls, and read browser console errors. Backend health alone is not playable evidence.
20. Treat scripts as production evidence only after executing syntax validation (`node --check`) and, where possible, the script against fresh supervised services. A script can be present, formatted, and covered by no tests while still being unusable (for example duplicate declarations or fabricated lifecycle success).
21. For this user's preferred Ponytail/full workflow, keep responses terse while working: act on the next blocker immediately, avoid repeated "waiting for review" updates when a completed result is available, and report only verified evidence after the final pass.
22. For world-first sandbox UI polish, prefer the smallest presentation-only pass before adding features: make the canvas primary, hide phase/prototype labels during Play, reduce persistent navigation to essential controls, preserve power/simulation/accessibility controls, then verify Home → Sandbox → Settings in a real browser with zero console errors. Commit the focused UI change only after focused build/typecheck and full gates pass; visual improvement cannot be inferred from build output alone.
23. After a completed review result arrives, read the full result if its summary is truncated, patch blockers immediately, and treat every subsequent tree edit as invalidating the review and gate claim. The required sequence is focused regression → fresh review → full gates → final git verification.

## Live-seam rules

### Replay

A replay is accepted only when all links exist:

- authoritative match/game-server producer records canonical inputs/events;
- the replay stores seed, config, initial snapshot, and ordered inputs;
- resimulation invokes a reducer whose output actually depends on seed/config/snapshot and inputs;
- terminal results are derived from authoritative state, not copied as stored truth;
- a regression mutates/removes stored derived events or varies the seed and proves the independent result changes or remains deterministic as appropriate.

Mapping inputs into events, returning stored arrays, or comparing two reads of the same Map is storage-shape evidence only.

### Post-match

Full-shape admin payload round-trips prove mapping, not authority. The terminal result/game-server seam must derive winner, score, rating, XP, mastery, mission, combat, graph, timeline, and recognition fields from authoritative events/telemetry. Do not fill missing fields with constants or zeroes merely to satisfy a type; wire the producer or mark the criterion incomplete.

### Security and operations

- Trust-boundary flags such as ranked privacy, release authorization, and streamer redaction must be enforced at the actual response/transport boundary, not merely returned as policy metadata.
- Recovery requires a producer that creates recoverable state, an idempotent transactional recovery method, and a live trigger plus a post-recovery read-path regression (for payments, the receipt itself must transition to granted with the final reward).
- Release rehearsal must observe authoritative allocation, authenticated release, downstream available state, and cleanup; a script that only reports `released: true` or probes health/metrics is insufficient.
- Accessibility/localization settings need runtime consumers. A state variable, CSS class without a rule, or locale dictionary with hardcoded rendered literals is decorative coverage.

## Evidence discipline

Keep implementation gates separate from PRD acceptance. Green format/lint/typecheck/tests/build prove code health, not authority, lifecycle, or integration. For the final report, list exact test counts, full-gate output, final review verdict, commit/push IDs only after verification, and deferred architecture separately from blockers.
