# New project orphans all users — existing logins break

Provisioning a NEW Supabase project and pointing `.env` at it does NOT carry
`auth.users` over — users live per-project. Symptom: every existing account gets
`invalid_credentials` on login even though the app loads, the web/auth APIs
respond, and env/keys are correct. Users report "เข้าไม่ได้" / "อีเมลหรือรหัสผ่านไม่ถูกต้อง".

## Diagnose in order

1. Auth alive check — expect `invalid_credentials` (400), NOT 401/404:
```bash
curl -s -X POST -H "apikey: $ANON" -H "Authorization: Bearer $ANON" -H "Content-Type: application/json" \
  -d '{"email":"x@y.z","password":"wrong"}' "$URL/auth/v1/token?grant_type=password"
```
`invalid_credentials` = auth works, the account just isn't in this project.

2. Confirm the project swap in git history:
```bash
git log --oneline -5 -- .env
git show <commit> | grep supabase.co   # old ref → new ref, e.g. svokkwqfrvgbokemyrow → ybxasyuhjgasnxkmemtu
```

3. Frontend build is fine (URL + anon key embedded correctly in dist JS) —
   it's purely a data-migration gap, not a code bug.

## Fix options

- (a) Users re-signup — fast, loses nothing but login (old data rows orphaned).
- (b) Migrate `auth.users` from the old project via SQL export/import — password
  hashes + identities + sessions; fiddly, test on a scratch project first.
- (c) Recreate users via Auth Admin API `POST /auth/v1/admin/users` with
  `email_confirm:true` — new passwords; keeps data rows keyed by user_id only if
  you reuse the old uuids.

## Real case (grider2, 2026-08-09)

commit `580ae23` "Provision new Supabase project (ybxasyuhjgasnxkmemtu)" swapped
the project; same-day user reported "เข้าไม่ได้". Everything else checked green
(web 200, React mounts, login page renders, Supabase health 401-ok). Root cause
was the project swap orphaning `auth.users`.
