---
name: production-regression-audit
description: "Use after deploys to re-audit sibling paths and live health."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [regression, production, audit, deployment, timezone, health]
---

# Production Regression Audit

Use after a production push, a multi-file bug-fix batch, or a request to
"check the bugs again". The goal is read-only evidence that the fix did not
leave sibling paths broken and that the deployed service is operational, not
merely buildable.

## Rules

- Read-only by default. Do not edit, commit, or redeploy unless explicitly asked.
- Findings need a file:line or live endpoint plus a root-cause explanation.
- A passing test suite is evidence, not proof that every sibling caller is correct.
- Separate code findings from operational findings (failed queues, stale jobs,
  unhealthy dependencies).
- Do not report style-only warnings as runtime bugs.

## Workflow

1. **Scope the change**
   - `git status --short --branch`
   - `git log --oneline -5`
   - `git show --stat HEAD` or diff the release range.
   - Confirm the working tree is clean before reporting deployed state.

2. **Run the tight baseline**
   - Type-check, focused regression tests, full tests, and production build as
     the repository provides them.
   - Record exact counts and exit codes. Do not reuse counts from an earlier
     commit.
   - If a build rewrites generated/index files, restore those files before
     calling the tree clean; never commit build artifacts unless the repo says
     they are source.

3. **Re-scan sibling paths**
   - Repeat the original bug grep across the entire repository, not only the
     files changed by the fix.
   - Trace every caller of the changed helper/service and inspect adjacent
     pre-existing logic for interactions that defeat the new guard.
   - Review schema/RLS/auth assumptions at the actual call sites.

4. **Probe boundary conditions**
   - For calendar logic, test a timestamp crossing the product timezone
     boundary. For Thailand, `2026-08-16T17:30:00Z` is `2026-08-17 00:30` in
     `Asia/Bangkok`; any local-date key should be `2026-08-17`.
   - Check date keys, day bounds, queue dedupe IDs, history labels, and weekly
     grouping separately; fixing one does not fix the others.
   - Prefer an existing timezone helper over ad-hoc `toISOString().slice(0, 10)`
     or server-local `setHours(0, 0, 0, 0)` for business-calendar logic.

5. **Verify the live boundary**
   - After deployment, query the real health endpoint and deployment status.
   - Report HTTP/DB/Redis/startup separately from queue status.
   - A `200` response with failed notification jobs is `operationally degraded`,
     not fully healthy. Treat queue failures as a separate finding and do not
     conflate them with a code regression without evidence.

6. **Report compactly**
   - Severity-ranked findings: Critical/High/Medium/Low.
   - Include file:line or endpoint, exact evidence, and why it matters.
   - Add a `Verified clean` section with tests/build/tree/deploy evidence.
   - State explicitly when no Critical finding was observed.

## Common high-value scans

```bash
# Remaining UTC date-bucket candidates
grep -RInE 'toISOString\(\)\.(split|slice)|setHours\(0, 0, 0, 0\)' src --include='*.ts' --include='*.tsx'

# URL-key and raw HTML security regressions
grep -RInE '\?key=|dangerouslySetInnerHTML|innerHTML' src supabase --include='*.ts' --include='*.tsx'

# Production health
curl -sS --max-time 20 https://<service>/health
```

See `references/timezone-and-live-health.md` for the reusable Thailand boundary
probe and evidence interpretation.

## Pitfalls

- Do not infer live cron schedules or deployment health from migrations, local
  tests, or a successful git push.
- Do not stop after the changed feature passes; sibling services often retain
  UTC date grouping or stale queue IDs.
- Do not silently fix findings during a requested review; report first and wait
  for scope to change.
