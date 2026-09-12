# Operational rehearsal reference

Use this after building `apps/api` and `apps/game-server`.

## Required evidence

1. Backup/restore each SQLite profile with positional arguments:

```bash
pnpm backup:sqlite data/godforge-economy.sqlite .tmp-review/economy-backup.sqlite
pnpm restore:sqlite .tmp-review/economy-backup.sqlite .tmp-review/economy-restore.sqlite economy
pnpm backup:sqlite data/godforge-operations.sqlite .tmp-review/operations-backup.sqlite
pnpm restore:sqlite .tmp-review/operations-backup.sqlite .tmp-review/operations-restore.sqlite operations
```

The restore command must report `integrity: "ok"`, required tables, and row counts. The scripts do not accept named flags; passing a literal `--` changes the source path and produces a misleading SQLite open error.

2. Start the executable entries, not the library module:

```bash
node apps/game-server/dist/main.js
node apps/api/dist/index.js
```

Use separate ports and disposable test values for `RANKED_ADMIN_TOKEN`, `STRIPE_WEBHOOK_SECRET`, and `STRIPE_PRICE_MAP` in local rehearsal. Never use production credentials in evidence.

3. Run the composed endpoint + lifecycle rehearsal:

```bash
API_BASE_URL=http://127.0.0.1:3108 \
GAME_SERVER_HTTP_URL=http://127.0.0.1:4000 \
pnpm release:rehearsal http://127.0.0.1:3108
```

It must check API health/readiness/metrics/dashboard, game-server metrics, then run guest → queue → allocation → result → authenticated release.

4. Run the operational safety rehearsal against an API configured with disposable test values:

```bash
RANKED_ADMIN_TOKEN=ops-test-admin \
STRIPE_WEBHOOK_SECRET=whsec_test \
STRIPE_PRICE_MAP='{"DIVINE_SHARDS_100":{"priceId":"price_test","amount":100,"currency":"usd"}}' \
pnpm ops:rehearsal http://127.0.0.1:3110
```

Expected result: `passed:true`, kill switch disabled/re-enabled, and signed payment webhook status `granted` or `pending`.

## Pitfalls

- A chained package command does not reliably forward one positional URL to every command; use explicit `API_BASE_URL` and `GAME_SERVER_HTTP_URL` environment variables.
- Restart services after rebuilding. A running `dist` process can expose stale metrics or release behavior.
- A load scenario that leaves a live in-memory match can make the next E2E fail; complete the result/release or restart both services.
- Runtime SQLite and review artifacts belong under ignored `data/`/`.tmp-review/`; inspect before cleanup and never delete without explicit consent.
