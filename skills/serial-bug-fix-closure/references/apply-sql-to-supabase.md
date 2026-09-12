# Applying SQL to Supabase when CLI paths fail

`supabase db push` may require elevated plan privileges (403 login-role errors like
"Your account does not have the necessary privileges"), and `supabase link` can fail
on API key schema quirks. The Management API query endpoint works with just an access token.

## Working method

1. Build a JSON payload with Python (raw SQL piped to curl fails — comments aren't JSON):
```python
import json
sql = open('migration.sql', encoding='utf-8').read()
json.dump({'query': sql}, open('payload.json', 'w'))
```
2. POST it with curl (python-urllib's default User-Agent gets Cloudflare-blocked,
   HTTP 403 code 1010; curl works):
```bash
curl -s -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST "https://api.supabase.com/v1/projects/<PROJECT_REF>/database/query" \
  --data-binary @payload.json
```
3. Empty response `[]` = DDL succeeded.

## Verify the change landed

Query `pg_proc.prosrc` and grep for new-logic markers:
```json
{"query": "SELECT prosrc FROM pg_proc WHERE proname = 'function_name'"}
```
Check the returned source contains your fix markers (e.g. `WHEN v_is_flat THEN 1`,
new scope lists, FOR UPDATE clauses).

## Notes

- Access token lives in `/d/hermes/.env` as `SUPABASE_ACCESS_TOKEN` on this machine;
  export it per-session.
- `supabase db push` without a link tries to create a login role → 403. Don't retry;
  go straight to the API method above.
