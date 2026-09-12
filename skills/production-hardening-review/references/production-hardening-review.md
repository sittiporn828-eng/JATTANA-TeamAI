# Production hardening evidence checklist

## Contract and runtime

- [ ] malformed JSON is retried then blocked
- [ ] missing required finding field is rejected
- [ ] `status: blocked|failed` cannot become completed
- [ ] ANSI-wrapped output is cleaned before parsing
- [ ] session IDs are parsed from real output formats
- [ ] real invocation and real resume use the same run ID/session
- [ ] upstream timeout/crash/503 is non-success
- [ ] provider usage is actual or explicitly `unavailable`

## Isolation and security

- [ ] source snapshot excludes harness control artifacts
- [ ] model workspace is disposable and source remains unchanged
- [ ] symlinks are rejected or safely dereferenced into the copy
- [ ] feature/prompt/session/path inputs reject traversal
- [ ] resume validates persisted workspace binding
- [ ] verification executes in a disposable copy; executable allowlist alone is insufficient
- [ ] patch validates workspace boundary, stale source, duplicate paths, and unexpected files

## Patch approval

- [ ] preview shows exact changed files
- [ ] explicit per-file allowlist is required
- [ ] exact preview patch hash is required for apply
- [ ] apply is atomic
- [ ] backup exists
- [ ] rollback is tested with an interrupted apply
- [ ] post-apply tests and diff review run before finalize

## Recovery and release

- [ ] state writes are atomic
- [ ] report/index/trace are durable before `reported`
- [ ] kill/restart tests cover Explore, Audit, Verify, and Report
- [ ] duplicate resume is idempotent
- [ ] concurrent resume locking/CAS is either implemented or explicitly a release blocker
- [ ] final compile + full test suite runs after the last edit
- [ ] independent read-only review has no unresolved Critical/High findings
- [ ] release report records passed checks and remaining blockers
