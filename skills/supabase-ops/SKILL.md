---
name: supabase-ops
description: Provision Supabase projects via CLI + Management API.
---

# Supabase Ops

Use when creating a new Supabase project (especially cloning schema from an existing project), pushing migrations, deploying edge functions, or resetting a project's DB password from the CLI.

## Setup

- Install: `npm install -g supabase` (works on Windows via hermes npm global)
- Login: `supabase login --token sbp_...` — the interactive browser flow FAILS in non-TTY shells ("Cannot use automatic login flow inside non-TTY environments"), so always pass `--token` or `SUPABASE_ACCESS_TOKEN`.
- PATs come from https://supabase.com/dashboard/account/tokens

## Create a new project

```bash
supabase projects create <name> --org-id <org> --region ap-southeast-1 --db-password "$(openssl rand -hex 12)"
```

- Flag is `--region`, NOT `-r` (CLI 2.x: "Unrecognized flag: -r").
- Get org id from `supabase projects list` output.
- If the password was generated inline, it is LOST — you must reset it (next section) before you can `db push`.

## Reset DB password (Management API)

```bash
curl -s -X PATCH -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" \
  -d "{\"password\":\"$NEWPW\"}" \
  "https://api.supabase.com/v1/projects/<ref>/database/password"
```

- Correct endpoint is `PATCH /v1/projects/{ref}/database/password` — NOT `/database-password` (that path 404s) and not POST/PUT.

## Get API keys

```bash
curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" "https://api.supabase.com/v1/projects/<ref>/api-keys"
```

Returns anon, service_role, plus the newer publishable/secret keys. `anon` is the one the frontend `VITE_SUPABASE_PUBLISHABLE_KEY` / client needs.

## Push migrations — CLI link bug workaround

`supabase link --project-ref <ref> -p <pw>` fails on CLI ≥2.112 with:
`failed to get api keys: SchemaError ... inserted_at` (CLI can't parse the new publishable-key format).

Workaround: skip link entirely, pass the DB URL straight to push:

```bash
DBURL="postgresql://postgres.<ref>:<pw>@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres"
supabase db push --db-url "$DBURL"
```

(Region hostname: `aws-0-ap-southeast-1.pooler.supabase.com`; 6543 = transaction pooler.)

## Deploy edge functions

Does NOT need `--db-url`; works with the project ref directly:

```bash
for fn in supabase/functions/*/; do supabase functions deploy "$(basename "$fn")" --project-ref <ref>; done
```

## Create test user + assign admin (no dashboard, no app signup)

Two Management-API calls cover "create a test account" end-to-end:

```bash
SR=$(curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  "https://api.supabase.com/v1/projects/<ref>/api-keys" \
  | node -e "let d='';process.stdin.on('data',c=>d+=c).on('end',()=>{const j=JSON.parse(d);console.log(j.find(k=>k.name==='service_role').api_key)})")

# 1. Create + confirm user via Auth Admin API (service_role key, NOT the Management PAT)
curl -s -X POST -H "apikey: $SR" -H "Authorization: Bearer $SR" -H "Content-Type: application/json" \
  -d '{"email":"admin@example.test","password":"<pw>","email_confirm":true,"user_metadata":{"nickname":"Admin"}}' \
  "https://<ref>.supabase.co/auth/v1/admin/users"   # -> {"id":"<uuid>",...}

# 2. Insert role directly via Management API SQL endpoint (201 = success, body = result rows)
curl -s -X POST -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" \
  -d '{"query":"INSERT INTO public.user_roles (user_id, role) VALUES ('"'"'<uuid>'"'"', '"'"'admin'"'"') ON CONFLICT (user_id, role) DO NOTHING;"}' \
  "https://api.supabase.com/v1/projects/<ref>/database/query"
```

- `email_confirm: true` → usable immediately (paired with `mailer_autoconfirm`). `.test` TLD emails work fine for test accounts.
- The `/database/query` endpoint runs arbitrary SQL with full privileges — the RPC-based `admin_set_user_role` is unnecessary here (it also guards on `auth.uid()` being admin, which service_role calls bypass awkwardly).
- Verify with a real password-grant login: `POST /auth/v1/token?grant_type=password` with the anon key → expect `access_token`.
- Save credentials to a local `TEST_CREDENTIALS.md` AND gitignore it (see auth-config reference); `/tmp` files die on reboot.

## Migrations from an OLD project (Lovable forks especially)

Expect these to fail on a fresh project and strip them BEFORE pushing:

1. **Hardcoded user seeds** — e.g. `INSERT INTO public.user_roles (user_id, role) VALUES ('<old-uuid>', 'admin')` fails with FK violation `user_id ... is not present in table "users"` because the old user doesn't exist in the new project. Remove the seed; assign admin via the app's RPC after first signup.
2. **`realtime.messages` policies** — the table was REMOVED from the Supabase platform; `ALTER TABLE ... ON realtime.messages` errors `relation "realtime.messages" does not exist`, and `realtime.topic()` errors `function realtime.topic() does not exist`. These RLS policies are obsolete (postgres_changes respects the underlying table's RLS). Replace with a comment noting they're no-ops.
3. Also check `storage.objects` policies reference buckets (`avatars`, `donation-slips`, ...). **Buckets themselves MAY already be created by migrations** if they contain `INSERT INTO storage.buckets` — don't assume they're missing. Verify with the Management API before telling the user to create them in the dashboard:

```bash
curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  "https://api.supabase.com/v1/projects/<ref>/storage/buckets" \
  | node -e "let d='';process.stdin.on('data',c=>d+=c).on('end',()=>{const j=JSON.parse(d);console.log(j.map(b=>b.id+'('+(b.public?'public':'private')+')').join(', '))})"
```

Only missing buckets need creation — via `POST /v1/projects/{ref}/storage/buckets` (Management API), not the dashboard.

## After provisioning

- Update `.env` (VITE_SUPABASE_PROJECT_ID, VITE_SUPABASE_PUBLISHABLE_KEY=anon, VITE_SUPABASE_URL) and `project_id` in `supabase/config.toml`.
- Verify: `vite build` succeeds and the new project ref appears in `dist/assets/*.js`.
- Dashboard-only steps (can't do from CLI): OAuth provider client IDs/secrets (LINE/Google — need provider credentials), custom SMTP. Storage buckets are NOT dashboard-only — verify/create via `GET|POST /v1/projects/{ref}/storage/buckets` (see migrations section).

## Edge function secrets (Management API)

```bash
# GET — list secret NAMES (values never returned)
curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  "https://api.supabase.com/v1/projects/<ref>/secrets"

# POST — body MUST be a JSON ARRAY of {name,value}, not an object
curl -s -X POST -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" \
  -d '[{"name":"CRON_SECRET","value":"<secret>"}]' \
  "https://api.supabase.com/v1/projects/<ref>/secrets"   # empty response = success
```

- Wrong shape (`{"name":...}`) → `"expected array, received object"`. Success returns empty body (exit 0).
- Secrets are read live by deployed functions — no redeploy needed after adding one.
- In git-bash, `curl -d @file` with an MSYS path fails; use `-d "$(cat file)"` or pipe `node ... | curl -d @-`.

## Manage pg_cron jobs via Management API SQL

Cron jobs are rows in `cron.job`; edit them through `/database/query` (same endpoint as arbitrary SQL):

```bash
# List all jobs with their target function + URL host
curl -s -X POST -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" \
  -d '{"query":"SELECT jobname, schedule, command FROM cron.job ORDER BY jobid;"}' \
  "https://api.supabase.com/v1/projects/<ref>/database/query"

# Replace a job's URL/headers: unschedule + re-schedule (returns [{"schedule":N}])
node -e 'console.log(JSON.stringify({query: `SELECT cron.unschedule('\''my-job'\''); SELECT cron.schedule('\''my-job'\'','\''0 3 * * *'\'', $$SELECT net.http_post(url := '\''https://<ref>.supabase.co/functions/v1/<fn>'\'', headers := jsonb_build_object('\''Content-Type'\'','\''application/json'\'','\''x-cron-secret'\'','\''<secret>'\''), body := jsonb_build_object('\''source'\'','\''cron'\''));$$);`}))' \
  | curl -s -X POST -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" -d @- \
  "https://api.supabase.com/v1/projects/<ref>/database/query"
```

- `cron.schedule()` third arg is the command text — wrap in `$$...$$` dollar-quoting (single `$` gets eaten by bash; build the JSON in node and pipe it to curl).
- `cron.unschedule('<name>')` takes the job NAME, not id.
- If only the URL/headers change, unschedule+schedule the SAME name — the job id changes but name/schedule are what matter.

## Pitfall: cron jobs pointing at an OLD project ref

After forking/migrating a project, `cron.job` commands keep the OLD project's function URL (e.g. `<old-ref>.supabase.co`) while the client env uses the new ref. Symptom: crons "run" (pg_cron fires) but the function never executes / 401s, or you deploy guards to the new project and the old-URL crons keep hitting the old project's unguarded functions.

Check every job's URL host against the ref your client uses:
```bash
curl -s ... -d '{"query":"SELECT jobname, command FROM cron.job;"}' ... | node -e "..."  # extract https://<ref>.supabase.co per job
```
Fix by re-scheduling each job with the new URL (above). Also verify the new project actually HAS the required secrets (LINE tokens, CRON_SECRET, AI keys) — migrated projects routinely miss them, and functions 500 with `... missing` until set.

## Edge function security checklist (verify_jwt=false especially)

- `supabase/config.toml` sets `verify_jwt = false` ONLY where needed (webhooks, crons). List them: `grep -B1 verify_jwt supabase/config.toml`.
- Every `verify_jwt=false` function needs its own auth: webhooks verify a platform signature (e.g. `x-line-signature` HMAC); cron-only functions MUST check a shared secret (`CRON_SECRET` env, header `x-cron-secret` or `Authorization: Bearer <CRON_SECRET>`). No guard = anyone can trigger it.
- Edge functions that use the SERVICE_ROLE key must re-verify the caller is who they claim: `getUser()` on a user client, then check `user_roles` for admin when the action crosses users (broadcast, targeting `user_ids` from the body). A regular user passing `user_ids: [<victim>]` to a service-role push function = spam/phishing vector.
- Test a deployed guard with curl: no-auth → expect 401, with secret → 200 (or a body showing it proceeded). Verify the deployed version bumped: `GET /v1/projects/<ref>/functions/<slug>` → `version` increments per deploy.

## Pitfalls

- CLI `projects create` output doesn't echo the DB password — always generate and save it yourself, or reset via PATCH.
- `supabase db push` applies migrations in a transaction per file; on failure it rolls back only that file — already-applied earlier migrations stay.
- `.env` is commonly committed in Lovable-generated repos (contains only anon keys, so low risk, but note it).
