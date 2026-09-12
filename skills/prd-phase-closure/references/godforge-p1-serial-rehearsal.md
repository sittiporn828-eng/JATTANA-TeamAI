# Godforge P1 serial rehearsal notes

Reusable lessons from P1 hardening work:

- Always pass the exact repo path to workers/reviewers: `C:\Users\Acer\Desktop\JATANA GROUP\Godforge`. A reviewer opening another repository is invalid evidence; re-dispatch with the exact path.
- Preserve serial closure: focused regression → full format/lint/typecheck/test/build/diff-check → fresh independent review. Any edit after review invalidates the review and gates; rerun the sequence.
- Backup/restore rehearsal safety: resolve source paths, copy both SQLite sources into a temporary disposable directory before any INSERT, invoke the real backup/restore helpers on copies, verify durable rows, and delete only the disposable tree. Never seed source/production DBs.
- Kill-switch rehearsal: prove admin-only mutation, authoritative queue and matchmaking rejection for already queued players, re-enable recovery/allocation, and audit entries.
- GM incident rehearsal: prove inspect → allowed status action → audit through the real admin API and console. Enforce lifecycle transitions server-side with a shared transition matrix; filtering UI options alone is not a guard. Return `404` for missing tickets and `409` for invalid transitions. Add a regression proving an illegal backward transition leaves durable state and audit invariants unchanged.
- Validate every rehearsal script with `node --check`; execute the script or a real route-level rehearsal before treating it as evidence.
