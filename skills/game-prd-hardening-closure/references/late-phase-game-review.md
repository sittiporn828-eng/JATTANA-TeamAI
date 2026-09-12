# Late-phase game review matrix

## Sequential execution

- Convert the blocker list into ordered work items with one item in progress.
- Finish each item with a focused test/build before touching the next item.
- Reconcile test counts after every item; a green full suite does not validate an untested new path.

## Cross-service allocation

- Trace API allocation request → returned ticket → authoritative server state → WebSocket envelope/input validation.
- Assert requested `match_id`, `seed`, and config are consumed, not merely echoed.
- Exercise: first allocation succeeds; duplicate allocation is rejected/idempotent; release returns the server to an allocatable state; stale client input is rejected; reconnect uses the same active ID.

## Security and abuse

- Use route-specific limits and test each required route class independently.
- Check both authentication and authorization: role/permission, actor ownership, confirmation, device/IP audit context, MFA boundary.
- Manual admin risk-flag insertion is not detection. Find the production caller that turns suspicious match patterns into a durable flag.

## Durable recovery

- Inspect schema and migration, then test two service instances against the same on-disk database.
- Recovery must be transactional, idempotent, and wired to a worker or operational route.
- Verify grant, ledger, receipt, uniqueness, and status transition together; rerun recovery and expect no duplicate reward.

## Client and release

- Trace a real event into notification/audio APIs and remove listeners on unmount.
- Check localization keys, long strings, text scale, contrast, keyboard/focus, and accessible names.
- A map CLI must fail closed on a bad fixture and be wired to ranked admission or explicitly run as a release gate.
- Run full gates after the last edit and obtain a new independent review; never reuse a verdict from before the last patch.

## Reusable rehearsal commands

For a Node/pnpm game monorepo with separate economy and operations SQLite stores:

```text
pnpm validate:migrations
pnpm validate:maps
pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build
node scripts/backup-sqlite.mjs SOURCE.sqlite BACKUP.sqlite
node scripts/restore-sqlite.mjs BACKUP.sqlite RESTORE.sqlite economy|operations
node scripts/load-smoke.mjs http://127.0.0.1:PORT 100 10
pnpm release:rehearsal http://127.0.0.1:PORT
```

Treat restore output as evidence only when it reports `integrity: ok`, required tables, and restored row counts. Treat load/release output as evidence only against a started service; a script run against a dead port is a failed rehearsal, not a passing check.

## Evidence classification

- `met`: exact criterion, real production caller/path, regression evidence.
- `partial`: a real slice exists but integration or required boundary is absent.
- `missing`: no implementation or only decorative/static behavior.
- `deferred`: broader production architecture outside the exact phase; record separately, never silently count as met.
