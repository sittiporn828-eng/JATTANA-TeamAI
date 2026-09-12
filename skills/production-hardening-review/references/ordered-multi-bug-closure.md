# Ordered Multi-Bug Closure

Use this when a confirmed backlog must be fixed in severity/order sequence.

## One-finding cycle

1. Read the finding and trace the real seam end to end.
2. Add one tight regression test that reproduces the exact symptom; run it RED.
3. Patch the shared root boundary with the smallest diff. Reuse existing helpers before adding abstractions.
4. Run the targeted test, nearest regression tests, and typecheck/lint for the touched scope.
5. Mark only that finding complete, record evidence, and start the next row.

## High-value boundary probes

- Calendar logic: freeze time and test a timestamp crossing the product timezone boundary; never infer correctness from host-local midnight.
- Async UI: resolve an older request after a newer request and assert the newer state wins.
- Duplicate side effects: exercise concurrent delivery and assert one side effect; claim with an atomic primitive (`SET NX`, unique constraint, or transaction).
- Optimistic UI: fail one concurrent mutation after another succeeds and assert rollback removes only the failed operation.
- Trust boundaries: reject invalid booleans, dates, and IANA timezones before persistence.
- Accessibility: prefer native `button`, `input`, or `checkbox` semantics over clickable non-interactive elements.

## Closure evidence

Run the full repository gates only after the last fix: tests, typecheck, lint, production build, and diff check. Restore generated artifacts before review. Distinguish code/test/build evidence from operational evidence; unavailable Redis, DB, or notification providers mean those flows remain unverified, not passed. Persist the result to the user's established durable notes before final delivery.

## Minimal-diff rule

Do not bundle unrelated cleanup, speculative abstractions, or broad refactors into a bug-fix batch. Remove dead code exposed by the fix, but otherwise touch the fewest files and keep the final report concise unless a detailed report was requested.
