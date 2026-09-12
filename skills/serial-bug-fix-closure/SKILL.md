---
name: serial-bug-fix-closure
description: "Use for ordered multi-bug fixes with regression gates."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [bug-fix, regression, tdd, timezone, concurrency, idempotency, release-gates]
    related_skills: [systematic-debugging, test-driven-development, production-hardening-review]
---

# Serial Bug-Fix Closure

Use when a user gives a confirmed backlog and requires severity/order sequence, one bug per cycle, and verified closure. This is a class-level workflow, not a feature-specific checklist.

## Operating rules

- Freeze the backlog before editing. Keep Critical/High/Medium/Low rows explicit.
- Close exactly one row at a time: understand → RED → minimal root fix → GREEN → nearest regression tests → mark complete.
- Re-read every changed caller and shared helper before patching; fix the common boundary rather than adding caller-specific guards.
- Reuse existing timezone, validation, persistence, and idempotency helpers. No speculative abstractions or unrelated cleanup.
- Keep code changes and evidence separate: passing tests/builds do not prove Redis, DB, queue, notification, or authenticated browser flows are operational.
- Persist important milestone results to the user's established durable notes before final delivery.

## Cycle

1. Record baseline branch, working tree, test counts, typecheck/lint/build status, and operational dependencies.
2. Read the finding and trace the actual seam end to end (UI → API → service → persistence → queue/notification where present).
3. Add the smallest regression test reproducing the exact symptom and run it RED. If no harness exists, document the missing seam and use the smallest deterministic check available.
4. Apply the smallest root-cause patch. Preserve user scoping, error handling, security, and accessibility.
5. Run the targeted RED→GREEN test, then related tests and typecheck. Do not start the next row until evidence is green.
6. Update the ledger/report/note with the exact result, limitations, and next row.
7. After the final row, run full tests, typecheck, lint, production build, and diff checks. Restore generated artifacts before reviewing status.

## Boundary probes

- Calendar/timezone: freeze time and test a product-local midnight boundary; never use host-local `setHours(0,0,0,0)` or UTC string slicing for business dates when a timezone helper exists.
- Async UI: resolve an older request after a newer request and assert the newest selection remains authoritative.
- Duplicate side effects: deliver concurrent identical requests and assert one side effect. Use an atomic claim (`SET NX`, unique constraint, or transaction) at the shared boundary; release a claim on failed persistence when safe.
- Optimistic state: fail mutation A after mutation B succeeds and assert rollback removes only A, not a whole snapshot.
- Trust boundaries: reject invalid booleans, dates, and IANA timezones before persistence with a clear client error.
- Accessibility: prefer native `button`, `input`, or `checkbox` semantics over clickable non-interactive elements.

## Final evidence

Report separate sections for implementation/tests/build and runtime operations. State unavailable dependencies explicitly; never call a flow operationally verified when readiness or queue infrastructure is failing. Do not commit, push, or deploy unless requested. See `references/closure-checklist.md` for a compact reusable checklist and evidence wording.

## Pitfalls

- A test count from before the final edit is stale.
- A passing unit mock can hide wire-shape or concurrency bugs.
- Fixing one date display without fixing its query/grouping boundary leaves sibling paths broken.
- Snapshot rollback is unsafe for concurrent optimistic mutations.
- Sequential Redis GET→DB INSERT→SET is not idempotency; the claim must be atomic.
- Build output can dirty tracked/generated files; inspect and restore deliberately.
