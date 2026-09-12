# Supabase Auth config via Management API (no dashboard needed)

`GET/PATCH /v1/projects/{ref}/config/auth` — the common auth settings that DON'T require the dashboard.

## Set site_url + redirect allow list (required for email flows on a new origin)

A fresh project defaults to `site_url: "http://localhost:3000"` and an empty allow list —
email confirmation and reset-password redirects silently break until this points at production.

```bash
curl -s -X PATCH -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" -H "Content-Type: application/json" \
  -d '{"site_url":"https://<app>.pages.dev","uri_allow_list":"https://<app>.pages.dev"}' \
  "https://api.supabase.com/v1/projects/<ref>/config/auth"
```

**Pitfalls:**
- `uri_allow_list` must be a **STRING**, not an array → 400 `"uri_allow_list: Invalid input: expected string, received array"`.
- Verb is **PATCH**. PUT → 404 (`Cannot PUT ...`).
- Response reflects the new values (`site_url`, `uri_allow_list`) when successful (HTTP 200).

## Email autoconfirm (instant signup)

```bash
curl -s -X PATCH ... -d '{"mailer_autoconfirm": true}' \
  "https://api.supabase.com/v1/projects/<ref>/config/auth"
```
- `mailer_autoconfirm: true` → new signups can sign in immediately, no confirmation link click.
- Recommended for consumer apps: Supabase's free email is heavily rate-limited, so forcing
  confirmation without your own SMTP bricks onboarding. Security tradeoff — keep false for sensitive apps.
- It's `mailer_autoconfirm` (not `EXTERNAL_EMAIL_ENABLED` — that's the legacy GoTrue env-var name).

## Read current state

```bash
curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  "https://api.supabase.com/v1/projects/<ref>/config/auth" | jq
```
Keys are camelCase: `external_email_enabled` (default true), `disable_signup` (default false),
`site_url`, `uri_allow_list`, `mailer_autoconfirm`, plus every OAuth provider's `external_<name>_enabled/_client_id/_secret`.

## What really is dashboard-only
- OAuth provider client IDs/secrets (LINE, Google, …) — need the provider's own credentials.
- Custom SMTP.
Everything else in this file is API-addressable.
