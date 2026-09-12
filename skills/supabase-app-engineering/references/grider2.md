# Grider 2 — project map & session notes

## Repo / deploy

- Clone: `C:\Users\Acer\Documents\GitHub\grider` (app name grider2)
- Push: `sittiporn828-eng/grider` via SSH key `work1` → GH Actions → CF Pages `grider2.pages.dev`
- `grider2` = `mediline2027/grider2` (HTTPS mirror). Supabase project `ybxasyuhjgasnxkmemtu` (was `svokkwqfrvgbokemyrow` before 2026-08-08 project swap — see supabase-ops `references/user-orphaning.md`)
- Android: appId `com.jatana.grider`, keystore `~/keystores/grider2-release.keystore`
- Stack: Vite + React 18, zustand persist (`grider-session-store`), @supabase/supabase-js, Capacitor 8, PWA, vitest (jsdom)
- Test include: `src/**/*.{test,spec}.{ts,tsx}`; setup `src/test/setup.ts`

## Domain

Rider income tracker (Grab/LINE MAN/FoodPanda etc.): `work_sessions` + children
`session_orders` / `session_tips` / `session_expenses` / `session_breaks` (all FK → work_sessions, `recorded_at` default now(), `id` default `gen_random_uuid()`), plus `maintenance_records`, `user_settings`, `fuel_prices`, `user_roles` (admin), emergency_alerts, push_subscriptions, notification_settings.

## Key files

- `src/pages/RecordPage.tsx` (~2,270 lines, monolith) — the whole live-session flow; 8+ insert sites, edit/delete handlers, stopwatch, voice orders
- `src/pages/AdminPage.tsx` / `TaxPage.tsx` / `SettingsPage.tsx` — other big pages
- `src/lib/exports/taxCalc.ts` — Thai progressive tax brackets (2567), pure
- `src/lib/pendingWrites.ts` — offline queue (see umbrella skill)
- `src/lib/dataExport.ts` — full JSON backup bundle (7 tables — `maintenance_items` does NOT exist)
- `supabase/functions/_shared/fuelSanity.ts` — pure sanity checks (no deno imports → vitest-safe)

## Edge functions (20)

admin-cleanup-inactive-users, admin-force-delete-user, ai-analyze (gemini-3.6-flash), broadcast-update, cancel-account-deletion, close-stale-sessions, delete-account, fetch-fuel-prices (EPPO scraper → Thai Oil API fallback), line-maintenance-alert (CRON_SECRET locked), line-webhook, morning-tip, process-account-deletions, send-emergency-alert, send-line-report, send-push, verify-slip, voice-parse-order (gemini-3.6-flash), zone-aggregate, zone-recommend, zone-trends.
CORS is `*` on all (auth-guarded; acceptable). All have auth except line-webhook (by design).

## Session 2026-08-09 — COMPLETE (verify ladder finished)

Done:
1. `?key=` → `x-goog-api-key` header on ALL 5 Gemini functions (ai-analyze, voice-parse-order, morning-tip, verify-slip, zone-recommend) — commit `2e76e3c`
2. EPPO sanity: `_shared/fuelSanity.ts` — band 5–100 THB/l, Δ vs yesterday > 3 THB → drop EPPO, fall back to Thai Oil API; both-broken → abort 500 (LPG hardcoded row moved AFTER abort check so the error path actually fires)
3. Offline queue `pendingWrites.ts`: insertQueued / flushPending / isNetworkError; wired 7 RecordPage insert sites + online-event flush + edit/delete guards; H1 race fixed — flushPending re-reads live queue entry per row (edit/delete mid-flush no longer lost)
4. `dataExport.ts` + "สำรองข้อมูล" QuickLink in SettingsPage (tone="sky" — no "violet" tone exists)
5. Tests: vitest 48/48 green (8 files); tsc --noEmit clean; prod build OK; eslint 445 pre-existing errors only

Commits on main: `dfeba05` (feature batch) → `2b1a533` (deploy retry) → `2e76e3c` (review fixes H1/M1/M3).

## ⚠️ OPEN: deploy stuck + login broken (2026-08-09)

- GH Actions → CF Pages did NOT deploy after 6+ hours (live still old asset hash `index-CgKw7rNC.js`; local build `index-DtSDeyTw.js`). Empty-commit retry didn't help. gh CLI token (mediline2027) INVALID → cannot read CI status; repo private. Suspects: CF API token expired in GH secrets, Actions disabled, or workflow failing silently. Next step: `gh auth login` / check repo Actions tab.
- "เข้าไม่ได้" (user login broken): root cause = Supabase project swap `580ae23` (svokkwqfrvgbokemyrow → ybxasyuhjgasnxkmemtu) orphaned ALL `auth.users`. App, web, auth API all respond fine; login = `invalid_credentials`. Fix options in supabase-ops `references/user-orphaning.md`. Wait — the project swap commit was on 2026-08-08 (11:27); user reported broken login 2026-08-09.
