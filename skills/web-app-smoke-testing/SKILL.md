---
name: web-app-smoke-testing
description: "Verify site logins fast; handles SPA click quirks."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [browser, testing, login, spa, qa]
    related_skills: [dogfood]
---

# Web App Smoke Testing (Login Verification)

Use when the user hands over site credentials and asks to "จำไว้ / ทดสอบเข้าไปใช้งาน" — verify a login works and the main app loads. Scope: credentials work + key screens reachable. Full exploratory bug-hunting belongs to the `dogfood` skill.

## Workflow

1. **Navigate to the site root** (`browser_navigate`). Landing page loads → good.
2. **Find the real login URL.** Clicking nav links on SPA sites often doesn't navigate. Extract actual hrefs instead of clicking blindly:
   ```js
   browser_console(expression="Array.from(document.querySelectorAll('a')).filter(a => /login|signin|เข้า/i.test(a.textContent)).map(a => ({text: a.textContent.trim(), href: a.href}))")
   ```
   Then `browser_navigate` straight to the matching href (e.g. `/auth`).
3. **Auth pages with role tabs**: a role/portal selector (staff vs admin) may appear *before* the login form. Click the correct tab first — the inputs render only after.
4. **Fill credentials** (`browser_type` on username + password fields), submit, then verify by snapshot: look for a welcome header naming the user, dashboard stats, or sidebar nav — proof of an authenticated session.
5. **Check Admin Panel / key routes** if present. JS-only buttons (no href) that "do nothing" on click → probe the route directly (`/admin`, `/dashboard`). If it loads, the bug is the button handler, not the route. Report it.
6. **Report concisely**: what passed, real numbers seen (proves data loads), and any dead buttons found.

## Pitfalls

- Clicking a "clickable" that leaves the URL unchanged = JS handler, not an anchor. Don't retry clicks; extract hrefs via console and navigate directly.
- Onboarding/welcome-tour modals can cover content after login — dismiss them ("ข้าม") or take a `full=true` snapshot to see what's behind.
- A `browser_snapshot` that returns almost nothing right after submit usually means the page is mid-transition — snapshot again (or full=true), don't assume failure.
- For local Vite smoke tests, verify the actual URL from the dev-server log rather than assuming forwarded CLI flags took effect; `pnpm --filter <app> dev -- --host ...` can pass a literal `--` to the script. If localhost works but 127.0.0.1 is refused in the browser bridge, navigate to the logged `http://localhost:<port>/` URL.

## Local full-stack smoke

For a local Vite/Node monorepo, do not assume the root `pnpm dev` script works. Confirm workspace filters first; if it reports no matching projects, start each app directly in separate terminals:

```bash
pnpm --filter <api-package> dev
pnpm --filter <game-server-package> dev
pnpm --filter <web-package> dev --host 127.0.0.1
```

Use the actual Vite URL printed by the dev server. Smoke the browser path in order: Home → Lobby → Start Match Preview → Play → select a power → Settings. Verify the accessibility/audio controls render, then inspect browser console for JS errors. Probe backend `/health`, `/ready`, `/metrics`, `/metrics/dashboard`, and game-server `/metrics`. Stop all dev processes after the test and report any root-script/startup issue separately from runtime bugs.

## Verification

- Login succeeded only when the post-login page shows user-specific state (name, data, menu) — not when it merely stopped showing an error.
- For a local app, a smoke pass requires a rendered key flow plus zero browser JS errors and healthy backend metrics, not just a successful HTTP page load.
- Credentials for the user's own sites live in memory; don't duplicate them in this skill.
