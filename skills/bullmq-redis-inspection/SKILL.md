---
name: bullmq-redis-inspection
description: "BullMQ queues via Redis: failed jobs, opts, retry fixes."
---

# BullMQ Redis Inspection

Debug BullMQ queues (MediLINE pattern) by reading Redis directly — no admin UI needed. Trigger: queue `failed` counts > 0 on a health endpoint, notifications not delivered, or verifying queue job options.

## Redis key layout

- `bull:<queue>:failed` — zset of failed job IDs (score = finish time). `zrevrange` for newest.
- `bull:<queue>:delayed` / `bull:<queue>:wait` / `bull:<queue>:active` — same pattern.
- `bull:<queue>:<jobId>` — hash: `data`, `opts` (JSON: attempts/backoff/repeat), `failedReason`, `attemptsMade`, `finishedOn`, `name`.
- `bull:<queue>:repeat` — repeatable-job metadata (next occurrence keys).

## Probe script

Use `scripts/check-failed-jobs.ts` (standalone ioredis, no app imports — avoids the app's env/connection baggage). Needs `REDIS_URL` env (fetch via `railway-deploy-ops` skill §3). Run: `REDIS_URL="$REDIS_URL" npx tsx scripts/check-failed-jobs.ts <queue1> <queue2> ...`.

## Clearing failed jobs

When failed jobs are stale (old 429s, pre-retry-fix data = wrong-time pushes), **delete, don't retry**. Use `scripts/clear-failed-jobs.ts` (standalone ioredis): deletes the `bull:<q>:failed` zset + each job hash. Run: `REDIS_URL="$REDIS_URL" npx tsx scripts/clear-failed-jobs.ts <queue...>`.

**Pitfall (hit 2026-08-08):** the default queue list (`medication-alert daily-summary meal-summary meal-nudge`) is NOT all queues in prod — `hydration-nudge` had 1 failed job the defaults missed, caught only via the app `/health` endpoint. Enumerate queues from `/health`'s `queues` key before clearing.

## Reading job opts

- `opts.attempts` / `opts.backoff` per job are baked in **at enqueue time**. Jobs enqueued BEFORE a queue-default change keep the old opts — only new enqueues get new defaults. Don't expect pre-existing delayed jobs to show the fix.
- BullMQ stores `attempts:0` in opts when no attempts were configured.

## Rate-limit retry pattern (LINE 429 etc.)

- Symptom: `failedReason: "429 - Too Many Requests"` when pushing many cards at once to one external API.
- Root-cause fix: set `defaultJobOptions: { attempts: 3, backoff: { type: 'fixed', delay: 30_000 } }` on the Queue constructor (already-proven pattern in the repo — copy it, don't invent).
- Audit ALL queues for missing retry — half the queues in the codebase had it, the failing one didn't.
- Do NOT retry stale failed jobs (yesterday's reminders/summaries = wrong-time data). They prune themselves via `removeOnFail`.

## Pitfalls

- Local `.env` REDIS_URL often points to localhost:6379 (dev) — connecting to it hangs/refuses; always pull the production URL for prod queue checks.
- `lazyConnect: true` + explicit `connect()` makes failures fail fast instead of hanging on retry loops.
- End scripts with `process.exit(0)` after `redis.quit()` — ioredis keeps the event loop alive otherwise.
