# Timezone and Live-Health Evidence

## Thailand boundary probe

A timestamp at `2026-08-16T17:30:00Z` is `2026-08-17 00:30` in
`Asia/Bangkok`. A local-date key, daily bucket, history label, and queue
same-day dedupe key should all resolve to `2026-08-17`.

Minimal shell probe:

```bash
TZ=UTC node - <<'NODE'
const instant = new Date('2026-08-16T17:30:00.000Z');
console.log({
  utcKey: instant.toISOString().slice(0, 10),
  expectedBangkokKey: '2026-08-17',
});
NODE
```

The probe demonstrates the trap; it is not a correctness test by itself.
Use the application's timezone helper or a small unit test to assert the
expected Bangkok key. Inspect sibling paths separately: health scoring,
weekly summaries, caregiver trends, vitals history, and medication queue IDs
may use different date logic.

## Live-health interpretation

After deploy, collect both deployment status and the real `/health` response.
A response with `startup: ready`, `database: up`, and `redis: up` proves those
boundaries only. If `notificationStatus` is `degraded` or `failedQueues` is
non-empty, report it as an operational finding even when HTTP is 200 and the
code tests pass. Do not call the release fully healthy until the queue issue
is understood or explicitly accepted.
