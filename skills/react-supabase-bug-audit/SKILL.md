---
name: react-supabase-bug-audit
description: Use when auditing a React+Supabase app for runtime bugs.
---

# React + Supabase Bug Audit

Systematic full-app audit for Vite/React/TS + Supabase apps (นัท's Lovable/Capacitor projects: grider2, meekamrai, MediLINE). Triggered by a "full app audit" prompt or a bug report from a real-device APK test.

## Baseline first (always run these before reading code)
```bash
npx tsc --noEmit          # type errors
npx vitest run            # existing tests
npm run build             # prod build (chunk-size warning = P3, ignore)
npx eslint src --ext .ts,.tsx | tail -30
```
- `no-explicit-any` errors are a repo-wide style pattern (hundreds) — P3, NOT runtime bugs. Don't "fix" them; don't report them as findings.
- `react-hooks/exhaustive-deps` + `react-refresh` warnings — usually harmless, skip unless a hook loop is suspected.

## The two bug classes that actually matter

### 1. UTC-vs-local date keys (Thailand UTC+7 = data silently lost)
Symptom: sessions/expenses from 00:00–06:59 land on the previous day; on the 1st of the month they vanish entirely; exports have a phantom first row and are missing the last day.

Root cause pattern — find with:
```bash
grep -rn "toISOString().split('T')\|toISOString().slice(0, 10)" src/
```
Day buckets are built in **local** time (`${year}-${month}-${d}`), but row keys use `toISOString()` (UTC). Fix once at the root: add a shared helper and use it everywhere:
```ts
// lib/constants.ts
export function localDateKey(d: Date | string): string {
  const dt = typeof d === 'string' ? new Date(d) : d;
  return `${dt.getFullYear()}-${String(dt.getMonth()+1).padStart(2,'0')}-${String(dt.getDate()).padStart(2,'0')}`;
}
```
Replace ALL `toISOString().split('T')[0]` day-bucket keys with it (monthly summaries, calendar, tax export day-fill loops, maintenance date filters). Query bounds (`gte/lt` on timestamps) can stay UTC — they're self-consistent. Only the **key generation** must match local buckets. Verify by grepping again — zero matches left except in comments.

**Thailand-specific check:** the helper above follows the device timezone. If the product’s calendar is defined as Thailand time regardless of a rider’s device timezone, implement the shared helper with `Intl.DateTimeFormat(..., { timeZone: 'Asia/Bangkok' }).formatToParts()` and assemble `YYYY-MM-DD`; test an instant such as `2026-02-10T18:30:00Z` (Feb 11 in Bangkok). Also scan `toISOString().slice(0, 10)` used for daily dismissal/notification keys and date-input defaults: it flips at 07:00 Thailand time.

### 2. Unhandled promise rejections → infinite spinner
Symptom: a page stuck on loading forever after a network drop / RLS error on device.

Find:
```bash
# Promise.all without .catch
for f in $(grep -rln "Promise.all" src/ --include="*.ts" --include="*.tsx"); do grep -q "catch" "$f" || echo "$f"; done
# .then() without .catch (hooks mostly)
grep -rln "\.then(" src/ | while read f; do c=$(grep -c "\.catch" "$f"); t=$(grep -c "\.then(" "$f"); [ "$c" -lt "$t" ] && echo "$f then=$t catch=$c"; done
```
Fix pattern: `.catch(() => { setLoading(false); /* empty state */ })` + a `cancelled` flag in the effect cleanup (prevents stale-response races when switching months/periods fast — setState after unmount). Supabase queries DO reject on network errors — don't assume `.then()` suffices. React-Query (`useQuery`) handles rejection natively — skip those.

**Also inspect fulfilled Supabase errors:** PostgREST/RLS/query failures commonly resolve as `{ data: null, error }`, so `Promise.all` and `.then()` still run. Never convert these to `data || []` for financial summaries or exports: check every response’s `.error` and throw/show an error before reading data. For a multi-table export, this is P1 because a generated file can look valid while silently omitting records.

### 3. Double-submit → duplicate rows in DB / double-paid API calls
Symptom: rapid double-tap on a save button inserts 2 identical rows (orders/fuel/expenses); AI analysis buttons fire the paid LLM call twice.

Find: buttons with async onClick and no `disabled`/busy state:
```bash
# async handlers with no guard
grep -rn "onClick={handle\|onClick={run\|onClick={send" src/pages/ src/components/ --include="*.tsx"
# buttons inside <form> missing type (default submit → accidental form submit)
grep -rn "<button" src/pages/ src/components/ --include="*.tsx" | grep -v 'type='
```
Buttons that are just close/back/cancel don't need guards — only handlers that write to DB or call paid APIs.

Fix pattern (one shared `submitting` state per page covers all adds):
```ts
const [submitting, setSubmitting] = useState(false);
const handleAddX = async () => {
  if (!user || !activeSession) return;
  if (submitting) return;          // guard FIRST
  if (!orderNote) { toast.error(...); return; }  // validation BEFORE setSubmitting
  setSubmitting(true);             // set only after validation passes
  const { data, error } = await supabase...;
  if (error) { toast.error(...); setSubmitting(false); return; }  // reset on error path
  ...success...
  setSubmitting(false);            // reset on success path
};
// button: disabled={submitting} + disabled:opacity-50
```
Critical: `setSubmitting(true)` goes AFTER all validation returns, or a validation failure leaves the button stuck disabled forever. Reset on EVERY return path (error + success).

### 4. Zustand persist + signOut → cross-user session leak (P1 privacy)
Symptom: user B logs in after user A logs out and sees A's active work session / orders / tips (data leaks across accounts on the same device).

Root cause: the store persists `activeSession` + session items to localStorage, but `signOut()` only calls `supabase.auth.signOut()` — the persisted session survives and is restored for the next user.

Fix (root — in the signOut function, not every caller):
```ts
const signOut = async () => {
  useAppStore.getState().resetSession();
  useAppStore.persist.clearStorage();   // zustand persist API
  await supabase.auth.signOut();
};
```
Also verify the page that consumes the session re-fetches from DB by user id on mount (`loadActiveSession` fetching `.eq('user_id', user.id)` — that covers token-expiry cases where user goes null without signOut). Keep the signOut fix anyway: it removes the stale-state window before the fetch resolves AND wipes personal data from the device.

### 5. Auth forms: empty email/password reach Supabase
Symptom: submit with empty/invalid email waits for a Supabase error instead of failing fast.
Fix: guard `email.trim()` + regex `^[^\s@]+@[^\s@]+\.[^\s@]+$` and `password` before `signUp/signIn/resetPassword` — with `setLoading(false)` on the guard returns.

### 6. Silent-fail writes: optimistic UI diverges from DB (P1/P2)
Symptom: "งานเสร็จแล้ว!" toast + share card show even though the `update` failed → user thinks the session ended, but it's still `active` in DB. Deletes: row disappears from UI but survives in DB (comes back on refresh) with no error shown.

Distinct from class 2 (crash/spinner) — this one FAILS SILENTLY: no crash, no toast, state just drifts. Find writes whose result is discarded:
```bash
# await supabase...update/delete/insert with no { error } destructure nearby
grep -rn "await supabase" src/pages/ src/components/ --include="*.tsx" | grep -v "error"
# also: result destructured but never checked
grep -rn "const { error" src/ | grep -v "if (error)" | head -20
```
Fix pattern — destructure and check before ANY optimistic UI update:
```ts
const { error: endErr } = await supabase.from("work_sessions").update({...}).eq("id", activeSession.id);
if (endErr) { toast.error("จบงานไม่สำเร็จ"); setEnding(false); return; }
setEnding(false);
// ...only now update UI / toast success
```
Applies to: session end, break toggle (2 updates — first can succeed, second fail = time math drifts), every delete button (orders/tips/expenses/breaks/history/maintenance), report deletes.
Corollary — local effect must happen even if DB write fails (e.g. push unsubscribe): wrap DB call in try/catch (log the error), then run the local `sub.unsubscribe()` unconditionally.

**Notifications:** do not optimistically mark alerts/announcements dismissed or replies seen without checking the write result and rolling back on failure. For notification preference reads, distinguish a successful missing row (default enabled) from `{ error }`; failed reads must fail closed so an opted-out user is not notified during an outage.

**Private Storage:** persist object paths (for example `userId/doc-id.jpg`), never a `createSignedUrl()` result. Generate signed URLs only when rendering; signed URLs expire and are bearer credentials.

### 7. RLS check BEFORE flagging missing `.eq('user_id')` (avoid false positives)
Queries that do `.update/delete().eq('id', x)` without a user_id filter LOOK like permission escalations but are safe when RLS is on. Verify first:
```bash
grep -rhE "CREATE POLICY.*(work_sessions|session_orders|session_expenses|user_settings|maintenance_records|profiles|fuel_price_reports)" supabase/migrations/*.sql
```
`auth.uid() = user_id` on USING/WITH CHECK for all operations = every `.eq('id')` write is scoped by RLS server-side. Only report it if a table's policy is missing or a policy lacks `auth.uid()`.

### 7b. Admin panel security (beyond RLS): SECURITY DEFINER RPCs must re-check role internally
Admin pages typically call `supabase.rpc('admin_*')` — RLS on tables is irrelevant here; the RPC body IS the gate. Client-side `isAdmin` is cosmetic. Verify in migrations:
```bash
grep -rn "admin_set_user_blocked\|admin_get_app_stats\|security definer" supabase/migrations/*.sql
```
Three things must hold, else it's a privilege-escalation hole:
1. RPC is `SECURITY DEFINER` (runs as owner, bypasses caller RLS) AND begins with `IF NOT public.has_role(auth.uid(), 'admin') THEN RAISE EXCEPTION` — no internal check = any authenticated user can call it.
2. `has_role()` itself is SECURITY DEFINER reading `user_roles` from DB (never trusts client-passed role).
3. `REVOKE EXECUTE ON FUNCTION ... FROM anon` (and ideally `authenticated` when only the RPC's internal check gates it).
Also watch for self-protection (`RAISE EXCEPTION` when `_target_user_id = auth.uid()`).

### 7c. XSS scan — one grep
`grep -rn "dangerouslySetInnerHTML\|innerHTML" src/ --include="*.tsx"` — shadcn `ui/` matches (chart.tsx) are safe; flag anything in app code rendering user input as HTML.

### 8. Dead-link check (routes)
Python heredoc comparing every `navigate('...')`/`to="..."` against App.tsx `path="..."` entries — catches nav targets that 404. Cheap and worth running once per audit.

### 8b. Dead-code check after route/feature removal
When a route is removed (e.g. WelcomePage skipped → straight to login), the page file + its private helper modules become dead code but stay imported-by-nothing and tsc still passes (they compile fine, just never render). Verify then `git rm`:
```bash
grep -rn "WelcomePage\|appFeatures" src/ --include="*.tsx" --include="*.ts" | grep -v "pages/WelcomePage.tsx"
# modules referenced ONLY by each other (comment mentions don't count) = dead
git rm src/pages/WelcomePage.tsx src/lib/appFeatures.ts
```
Re-run tsc after deletion — zero refs left means the build is still clean.

### 9. Supabase Edge Functions audit (P0 territory — run LAST, don't skip)
Frontend RLS is table-level, but edge functions run with `SERVICE_ROLE_KEY` = **they bypass RLS entirely**. A function that accepts user-supplied targeting is a spam/phishing hole unless it re-checks authorization in its own body. grider has 19 functions / 3,600 LOC.

Map them first:
```bash
ls supabase/functions/
grep -B1 "verify_jwt" supabase/config.toml   # which are public (no JWT gate)
# auth-coverage scan: 0 = cron-by-design OR a hole — inspect each
for fn in supabase/functions/*/; do echo "== $fn"; grep -cE "getUser|Authorization|has_role|CRON_SECRET" "$fn"index.ts; done
```
The two P0 bugs found this way (both fixed):
- **`send-push` accepting `user_ids` from body with service role** → any logged-in user could push spam/phishing to arbitrary users (key check was bypassable: unknown key → `setting = null` → passes). Fix: if `user_ids` targets others, require `has_role(admin)` via service-role query of `user_roles`; non-admin → 403. Legit admin broadcast sends no `user_ids` (null = all) so it's unaffected.
- **`verify_jwt = false` function with no in-body guard** → `line-maintenance-alert` (pg_cron job, sends LINE push to every opted-in user) was publicly callable = LINE spam vector. Fix: require `Authorization: Bearer ${CRON_SECRET}` like `close-stale-sessions`/`process-account-deletions`/`fetch-fuel-prices` already do.

Guard triage for every function (all three patterns exist in this codebase):
- Webhooks (`line-webhook`, `verify_jwt = false`): verify signature (HMAC `x-line-signature`) — signature IS the auth.
- Cron jobs: `CRON_SECRET` header check (or `verify_jwt = true` + user-triggered like `morning-tip`).
- Admin ops (`broadcast-update`, `admin-*`): `has_role(auth.uid(),'admin')` via service-role query + self-protection (`target === caller → raise`).
- Client data ops: `getUser()` JWT + scope queries `.eq('user_id', user.id)` (verify-slip does this — reads slip/amount from DB, never trusts client values).
**9b. pg_cron schedule verification — check LIVE first, migrations are NOT proof**
A cron-guarded function is dead weight if nothing schedules it. `CREATE EXTENSION pg_cron` proves nothing. **But committed migrations also prove nothing** — users routinely create pg_cron jobs in the Dashboard SQL editor (grider live had line-maintenance-alert + correct URLs long before the repo did). Audit the LIVE DB before concluding anything:
```bash
TOKEN=$(sed -n '2p' <tokenfile> | tr -d '\r\n ')   # 2-line token file: line 2 = sbp_ token
curl -s -X POST "https://api.supabase.com/v1/projects/<ref>/database/query" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"query":"select jobid, jobname, schedule, left(command,220) as cmd from cron.job order by jobname"}'
```
This Management API endpoint works WITHOUT `supabase link` (which can fail with a CLI SchemaError) — inspect `cron.job` AND run arbitrary SQL (create/unschedule jobs). For SQL with quotes/dollar-quotes, shell quoting breaks: build the JSON with python `json.dumps({"query": sql})` into a file, then `curl --data-binary @file`.
Then for each job check:
1. **`net.http_post` URL matches the CURRENT project** — migrations copied when provisioning a new Supabase project carry the OLD project's URL (`svokkwqfrvgbokemyrow.supabase.co` vs the real ref in `.env`/config.toml). Jobs silently hit the wrong project: account-deletion purge never runs (privacy promise broken), zone aggregation never runs. Fix = the migration URL, not the function.
2. **Header secret FORMAT matches what the function checks** — `close-stale-sessions` reads `x-cron-secret`, `line-maintenance-alert` reads `Authorization: Bearer CRON_SECRET`. Format mismatch = 401 on every run.
3. **Every cron-guarded function has a live schedule entry** — cross-check the function list against the live `cron.job` result, not against migrations. Only the genuinely-absent job is a finding (grider: close-stale-sessions was missing live; line-maintenance-alert existed at 0 8 * * *).
Adding a missing job: **reuse the secret value from an existing live job** (`left(command,...)` shows it in `x-cron-secret`/`Bearer`) — it must equal the edge function's CRON_SECRET env exactly; don't invent one. Also flag hardcoded JWT placeholders in migration headers (secret hygiene — check git history for full tokens).
Admin dashboards often expose manual "run now" buttons for these — that's a mitigation, not a fix.

**Recommended cron-job auth pattern (fix migrations with):** source the secret from Supabase Vault inside the job command — `headers := jsonb_build_object('x-cron-secret', (select decrypted_secret from vault.decrypted_secrets where name = 'CRON_SECRET'))` (or `'Authorization', 'Bearer ' || ...` for Bearer-guarded functions) — with a `raise notice` fallback placeholder so a fresh provisioning fails closed (401) instead of hardcoding a token. One-time setup: `select vault.create_secret('<value equal to the edge function CRON_SECRET env>', 'CRON_SECRET');` — the value lives only in the dashboard, never in the repo. Also qualify `extensions.net.http_post` (pg_net installed `WITH SCHEMA extensions` isn't on cron's search_path — unqualified `net.http_post` never resolves).

**Deploy edge functions without `supabase link`:** if `supabase link --project-ref <ref>` fails with a CLI SchemaError (e.g. "Expected a string matching the RegExp ... inserted_at" — CLI can't parse the api-keys response), deploys still work: `SUPABASE_ACCESS_TOKEN=<sbp_ token> supabase functions deploy <name> --project-ref <ref>`. `db query` is also reachable without link via the Management API (see 9b). Token files may be 2-line (comment header + token) — extract line 2, strip whitespace/CR.

**Deploy gotcha:** fixing an edge function in the repo is NOT deploying it — Supabase functions live server-side. After pushing code changes you must run `supabase functions deploy <name>` (or all) per function touched, else the fix has no effect in prod. This bit us: two security fixes were committed+pushed but not deployed.

## Commit/diff review mode (read-only, no build)

Triggered by "review diff ของ commit X" / "review X vs Y" — same bug-hunting, scoped to a commit instead of the whole app. No edits, no build; still verify claims with git + grep.

1. **Scope first**: `git log --oneline -5` (which commit is HEAD), `git diff --stat <base> <head>`, then `git diff <base> <head> -- <files>`.
2. **Read the FULL current files, not just hunks** — a hunk can't show the caller's error handling or a pre-existing block a few lines below that neutralizes new logic. Trace every caller of each changed function.
3. **Verify every schema assumption against migrations**: ORDER_COLUMN/TS_COLUMN names, RLS policies (`grep -rn "<table>" supabase/migrations/`). A typo'd column = PostgREST 400 = whole feature dead (e.g. export `.order("created_at")` on a table without it).
4. **Check NEW-vs-PRE-EXISTING interactions**: new code setting `allPrices = []` to abort was defeated by a pre-existing hardcoded LPG push a few lines later (`if (allPrices.length === 0)` never fires → writes 1 hardcoded row + returns success). Grep the surrounding logic the diff left untouched.
5. **Check sibling call sites for incomplete fixes**: one function fixed → `grep -rn "?key=" supabase/functions/` found 3 more leaks (morning-tip, verify-slip, zone-recommend).
6. **Skim the tests**: do they cover the claimed behaviors (queued id preserved, permanent drop, offline queue)? Note what's NOT tested (flush-vs-edit race, concurrent flush) and check those paths manually.
7. **Report format**: severity-ranked findings (CRITICAL/HIGH/MEDIUM/LOW) with file:line + one-line rationale, plus a "verified clean" section listing what passed (RLS, dedup, schema, auth). State no-CRITICAL explicitly if so. Offline-queue failure modes live in the `supabase-app-engineering` skill.

## Audit workflow — subagent vs self

- For big codebases (150+ files / 30K+ LOC) fan out 3 parallel leaf agents: pages / hooks+lib / components. Give each: exact dir, what to check (unhandled null, async errors, races, loops, nav targets, missing loading states, form validation gaps, double-submit, missing type="button"), severity scale (P0 crash/P1 functional/P2 UI/P3 minor), "report ONLY real bugs with file:line + snippet + root-cause fix", "read-only, do NOT modify".
- **Known failure: parallel subagents hit HTTP 429 rate limits on shared API keys** — results come back as "API call failed after 3 retries". Fallback ladder: (1) re-dispatch the failed zones SEQUENTIALLY (one delegate_task call), (2) if that also 429s, audit yourself with the grep patterns above + read the biggest/riskiest files (RecordPage/AdminPage-size pages, hooks with Promise.all). The grep patterns catch most real bugs without an LLM anyway.
- **In practice on this machine, subagent audits failed 5× in a row with 429** (both parallel and sequential, even with 30-60s pauses) — plan for self-audit via grep as the PRIMARY path for large passes; treat subagent success as a bonus, not the plan.
- Use a tiny python heredoc for checks bash grep can't express (buttons inside `<form>` blocks, async handlers lacking guards): walk `src/`, regex `<form[^>]*>(.*?)</form>` then `<button([^>]*)>` without `type=`, report file:line.
- Read the subagent's saved summary file — delegation results are truncated in-chat; `read_file` the `subagent-summary-*.txt` path for the full report.

### Full-app audit deliverable (นัท's required format)
When the prompt is "ตรวจโค้ดและ Flow ทั้งหมดทีละฟีเจอร์" / "full feature-by-feature audit": write **`AUDIT_REPORT.md` at the repo root** (read-only — no code changes until told). Required structure, per feature:
1. ไล่ Flow ต้นจนจบ, 2. ตรวจ Frontend/Backend/API/Database + การเชื่อมโยง, 3. Bug/Logic Error/Flow ที่ขาด/Edge Case, 4. ไฟล์+บรรทัด, 5. Severity: **Critical / High / Medium / Low** (Thai labels), 6. ตรวจทีละฟีเจอร์จนจบก่อนเริ่มถัดไป, รวบรวมทั้งหมดในรายงานเดียว
Report skeleton: severity summary table (0-Critical count) → feature-by-feature findings (each: flow, severity, file:line, root-cause fix) → "verified clean" section → file:line lookup table → baseline results → extra notes (secrets hygiene, e.g. TEST_CREDENTIALS.md with real DB passwords). Start baseline (tsc/vitest/build) in background while reading code. End by logging to `02 Daily/<date>.md` (สิ่งที่ทำ/Review/Verify) — even for read-only audits.

## After fixes (verify everything, per user's ECC rule)
1. `npx tsc --noEmit` — must be clean
2. `npx vitest run` — existing tests must pass (18/18 for grider)
3. `npm run build` — prod build OK
4. `git diff --name-only | grep -E "\.(ts|tsx)$" | xargs npx eslint` — no NEW errors vs pre-existing any-pattern
5. Review full `git diff` for unrelated changes
6. Rebuild APK: `cd android && ./gradlew.bat assembleDebug` (after manifest changes), send to user for real-device test
7. Commit + push (grider: `GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519_github_work1" git push origin main` → GH Actions auto-deploys CF Pages)
7b. If edge functions were touched: `supabase functions deploy <name>` per function (repo push ≠ server deploy — ask user or run CLI)
8. Log to `02 Daily/<date>.md` in the user's Obsidian vault (สิ่งที่ทำ/Review/Verify)

## Android-specific audit (when user tests via APK)
- Web APIs in Capacitor WebView need manifest permissions the web app never declared: `getUserMedia` → RECORD_AUDIO, `geolocation` → ACCESS_COARSE/FINE_LOCATION, `vibrate` → VIBRATE. Check `grep -rn "getUserMedia|geolocation|vibrate" src/` then add `<uses-permission>` to `android/app/src/main/AndroidManifest.xml`.
- Routes reachable while logged out (password reset links) must be handled in the `if (!user)` guard, not inside AppLayout: Supabase appends `#access_token=...&type=recovery` — listen for `PASSWORD_RECOVERY` in `onAuthStateChange` + `getSession()` then `updateUser({ password })`.
- **Don't rebuild+reinstall APK for every code change.** Offer the ladder: (1) PWA first — deployed web build (`grider2.pages.dev`) + Add-to-Home-Screen covers ~90% of UI/logic/audio/GPS testing with zero installs (service worker auto-updates on relaunch); (2) Capacitor live reload for real native testing — `server: { androidScheme: "https", url: "http://<lan-ip>:5173", cleartext: true }` in capacitor.config.ts + `npx cap sync` once + one APK install, then edits hot-reload into the WebView; (3) full rebuild only for manifest/native-wrapper changes. Revert the `server.url` block before release builds.

## Pitfalls
- The `patch` tool's "lint: error TS6053 file not found" on .ts edits is a tool path quirk, not a real error — verify with full `npx tsc --noEmit` instead.
- **Patch tool double-escapes backslashes in regexes** — editing Deno/TS code that contains regexes (`\s`, `\d`, `\n`) via the `patch` tool can write `\\s` into template-literal regexes (`new RegExp(\`...\\s+\d+\`)` matches a literal backslash, silently breaking the pattern). After any regex edit, verify actual file bytes with `sed -n '...'` — never trust the diff display. (Bit us on the line-webhook order-parse regex.)
- Don't trust a single failing test run on slow HDD — rerun once before treating it as a regression.
- Debug APK builds fast after first run (~4s incremental); AAB needs `npm run build && npx cap sync android` first.
