---
name: supabase-app-engineering
description: "Supabase: offline queues, edge functions, vitest."
---

# Supabase App Engineering

Class-level patterns for นัท's Supabase-backed React apps (grider2, meekamrai, MediLINE LIFF). Client-side patterns + edge function (Deno) development/testing. Project-specific maps live in `references/` (e.g. `references/grider2.md`).

## Offline write queue (client-uuid pattern)

Core trick: **generate the row id client-side with `crypto.randomUUID()` and send it in the insert payload.** Postgres `gen_random_uuid()` default only applies when `id` is omitted — the server row gets the SAME id. No remap needed on flush. Works because child rows (orders/tips/expenses/breaks) reference a session id that already exists server-side (sessions are started online).

Flow (see grider2 `src/lib/pendingWrites.ts`):
1. `insertQueued(supabase, table, payload)`: if `navigator.onLine === false` → build row `{...payload, id: crypto.randomUUID(), recorded_at: new Date().toISOString()}`, push to localStorage queue, return it (UI shows it instantly). If online → try insert; on network-ish error → same queue path. Non-network errors (FK 23503 etc.) pass through — session-expired handling must still work.
2. `flushPending(supabase)`: FIFO re-insert each queued row (id already set). Network fail → keep for next retry; permanent error → drop (row can never sync).
3. Edit/delete of a still-pending row must mutate the queue entry, not supabase (row doesn't exist server-side yet) — guard each edit/delete handler with `isPending(id)`.
4. Wire flush to `window 'online'` event + on mount; toast synced count.

### Failure modes to check when reviewing/extending the queue
- **Snapshot-read race (worst, silent data loss)**: `flushPending` reads the queue ONCE (`for (const w of getPendingWrites())`) then awaits one network round-trip per row. An edit (`updatePendingWrite`) or delete (`dropPendingWrite`) landing mid-flush is lost — flush inserts the stale snapshot copy, then `dropPendingWrite` removes the entry that held the edit (DB=old value, UI=new); a deleted row gets re-inserted into the DB. Fix: re-read the live entry per row before inserting; skip if gone.
- **Ambiguous-commit dedup is what makes the pattern safe**: network drop AFTER server commit → row queued with its client uuid → flush re-inserts same id → PK violation 23505 → permanent → dropped. Never remove client-id-in-payload; without it a flush retry duplicates the row.
- **`isNetworkError` semantics**: network errors have NO `code` (message regex `/fetch|network/i`); ANY `code` is a real Postgres/PGRST error. If it returns true when `code` is set, FK 23503 gets queued and the session-expired toast never fires. Auth failures (PGRST301 etc.) match neither network nor permanent codes → row kept for retry — correct.
- **Permanent vs transient on flush**: drop only for 23503/23505/22xxx/42xxx; keep everything else. Prefer an explicit list over a broad regex — `3[0-9A-F]` also matches 38xxx external-routine codes.
- **`data ?? row` fabricated fallback**: `insertQueued` returns the client row as `data` even when a non-network insert errored. Every caller MUST check `error` before using `data`, or a fake row enters local state.
- **Silent FK drop = silent data loss**: queued rows referencing a session deleted server-side get 23503 at flush → dropped with zero feedback after the "รอซิงก์ 📡" toast. Flush should report dropped counts.
- **Side-table writes must be in the queue too**: break flow updates `work_sessions.status`/`total_break_duration_ms` without queueing → DB/UI drift (ghost "paused" state, wrong break totals) after offline breaks.

## Testing edge function logic with vitest

Deno edge functions import from `https://...` — vitest (node) cannot resolve those. Fix: put pure logic in `supabase/functions/_shared/<name>.ts` with **zero deno imports**, then vitest imports it via relative path from `src/test/`. Only the thin `index.ts` wrapper keeps deno imports. (`_shared/` is the Supabase convention for code shared across functions; it's auto-excluded from deployment.)

## Gemini API key — never in query string

`?key=${KEY}` leaks the key into URLs/server logs. Use header `x-goog-api-key: <key>` instead. Applied in grider2 `ai-analyze` + `voice-parse-order`.

**When fixing one function, grep the whole functions dir for the same leak** — `grep -rn "?key=" supabase/functions/`. As of the offline-queue commit, `morning-tip`, `verify-slip`, `zone-recommend` still pass the key in the URL; a one-function fix leaves the security hole open elsewhere.

## E2E-encrypted sync (zero-knowledge — server can't read data)

When a local-first app must sync across devices but the developer/server must NOT be able to read user data (privacy selling point), use **E2E encryption**: encrypt on-device with a key derived from the user's **passphrase**, upload only the ciphertext (e.g. to Supabase Storage), and let another device decrypt with the same passphrase. Server holds unreadable blobs only. Full working lib + test in `references/e2e-encrypted-sync.md`.

Key facts to get right (all bit me):
- **TS 5.6+ `Uint8Array<ArrayBuffer>` vs `Uint8Array<ArrayBufferLike>`**: WebCrypto params (`BufferSource`) reject `ArrayBufferLike`. `Uint8Array.from(...)` returns `ArrayBufferLike` → compile error "Type 'ArrayBufferLike' is not assignable to type 'ArrayBuffer'". Build arrays with `new Uint8Array(n)` + a `for` loop (typed `Uint8Array<ArrayBuffer>`), not `Uint8Array.from`.
- **Don't spread a `Uint8Array`** (`String.fromCharCode(...arr)`) — fails under `downlevelIteration`/ES5 target. Use a `for` loop to base64.
- **Pass every typed-array param that flows into WebCrypto as `Uint8Array<ArrayBuffer>`** (e.g. the PBKDF2 salt) or the same error propagates.
- **Node 22 already has `globalThis.crypto`** (getter-only) — do NOT `globalThis.crypto = webcrypto` in tests (TypeError). WebCrypto `crypto.subtle` is available in both browser and Node 22 out of the box.
- **Exclude `*.test.ts` from the build tsconfig** so `tsc -b` doesn't demand `@types/node` (tests use `process`/node globals); run tests with `tsx` instead.
- Cost of zero-knowledge: **lost passphrase = unrecoverable data**. UI must warn this clearly + offer a passphrase backup path.

## Pitfalls

- **read_file reports CRLF files as binary** (Windows git checkout, core.autocrlf): `file` says UTF-8 but read_file refuses. Read via `python -c "import io; print(io.open(path, encoding='utf-8').read())"`. Note: นัท's Obsidian vault `50 Wiki/SCHEMA.md` and `index.md` are UTF-16 (BOM) — decode via python `d.decode('utf-16')`; `log.md` is UTF-8. Wiki rule still requires reading SCHEMA + index + log before wiki edits.
- **tsc TS6053 "file not found" noise on patch** of `supabase/functions/*`: expected — Deno files are outside the tsconfig include. Not a real error; ignore.
- **Vitest + jsdom**: `crypto.randomUUID` doesn't exist in jsdom → `vi.stubGlobal("crypto", { randomUUID: () => "stub" })`. Async helpers must be awaited; an `it("...", () => {...})` callback with `await` inside needs `async`.
- **Lint-delta check**: `git stash -q && npx eslint <files> | grep -cE "error"; git stash pop -q` and compare counts — tells you whether YOUR edits added errors vs pre-existing noise. Edge functions carry hundreds of pre-existing `any` errors; don't chase them.
- **Verify before declaring a table exists**: `grep -rn "<table>" supabase/migrations/` — grider2 has no `maintenance_items` table despite being referenced in some queries. Export lists must only include real tables.

## Verify ladder (ECC)

vitest run → eslint (delta only) → tsc → build → subagent review → commit/push → 02 Daily log. Session got cut mid-verify in the first grider pass — resume from the ladder, don't redo completed items.

## CI migration validation without local Docker

When local Docker/virtualization is unavailable, validate the migration in GitHub Actions instead of inventing a local substitute: add a `postgres:16-alpine` service with a health check, bootstrap only the Supabase-specific `auth.users` dependency for vanilla PostgreSQL, run `psql -v ON_ERROR_STOP=1 -f supabase/migrations/*.sql`, then query `to_regclass` for required tables. Build API/Game Server images on the same Ubuntu runner. This verifies the real migration and container build paths while keeping the local machine unchanged.

## Bulk inventory import for Supabase-backed POS apps

When importing a large ingredient catalog into meekamrai, target `public.inventory_items` for real stock management—not the legacy `public.ingredients` library used by `/ingredients`. The `/admin/inventory` page currently supports single-item entry; a CSV importer is the minimum useful extension.

Recommended import contract: UTF-8 CSV with `name,unit,category,current_stock,unit_cost,min_stock,sku,notes`. Preview and validate before insert; normalize duplicate spellings (for example Thai typo variants of โชยุ), report duplicate/invalid rows, and require an explicit skip-or-update choice. Set `user_id` from the authenticated user and `branch_id` from the active branch; default initial stock/cost/minimum to zero rather than inventing values. Use a bounded batch insert and invalidate the `inventory_items` query afterward.

Keep raw inventory separate from prepared sub-recipes. Store ingredients such as โชยุ, มิริน, garlic, and ramen noodles as `inventory_items`; model น้ำซุปโชยุ, ซอสยากิโซบะ, and curry sauce as recipes that consume those rows, otherwise stock is double-counted. Do not claim an import is complete until a read-back confirms inserted count, duplicate handling, user/branch scope, and unit/category values.

## References

- `references/grider2.md` — repo map, edge function list, session work + pending verify items.
- `references/e2e-encrypted-sync.md` — working WebCrypto E2E-encryption lib + test + sync architecture (zero-knowledge cross-device sync).
