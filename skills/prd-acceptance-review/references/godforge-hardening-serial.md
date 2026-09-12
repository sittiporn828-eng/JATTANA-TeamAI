# Godforge Phase 13–16 hardening notes

Use this as a compact evidence checklist, not as a substitute for reading the current PRD.

## Proven patterns

- Game-server `/release` must be authenticated in production and bound to the active `match_id`; API must send both the token and binding.
- Separate SQLite databases need separate backup/restore profiles. A restore rehearsal must copy a disposable backup, run `PRAGMA integrity_check`, verify required tables, and verify non-zero sample row counts.
- Pending-payment recovery is only real when a production callback can create `status='pending'` and a startup/timer/manual trigger can transition it to `granted` idempotently.
- A release rehearsal script must run against a started service. Health/ready/metrics checks are useful smoke checks but do not prove incident, restore, allocation, or compensation workflows.
- Determinism evidence should hash checkpoint state with identical seed/config/input. A create-world-only test is weaker than an actual match contract test.
- UI accessibility settings must affect runtime behavior. CSS-only classes or a test-only browser event are partial evidence.

## Verification commands

```sh
pnpm validate:migrations
pnpm validate:maps
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
git diff --check
```

For operational scripts, start the built service first, then run `pnpm release:rehearsal URL`, `pnpm load:smoke URL REQUESTS CONCURRENCY`, and both economy/operations backup-restore profiles with disposable files.

## Common false positives

- Green unit tests do not prove cross-service allocation/release.
- A `/metrics` endpoint with counters is not automatically a dashboard or alert system.
- A recovery method with no producer is dead-path code.
- A locale selector with many hardcoded strings is not runtime localization coverage.
- A simulation replay hash is not full competitive-match determinism unless the match input/config contract is exercised.
