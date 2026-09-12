# Godforge Phase 12 — GM console / LiveOps closure

Blocker class (from a fresh fail-closed review of a first implementation):

1. **GM auth**: shared static bearer-token comparison ≠ login acceptance. Real fix = `POST /admin/login` (bootstrap token as credential) issuing opaque session tokens with expiry + `/logout`, and admin routes validating the session map. Keep the static token only as the login credential.
2. **Read models must serve REAL data — a stub is not acceptance.** `matches: []` plus a `note:` field is a stub and the reviewer will not count it. Every GM monitor endpoint needs a live source:
   - online players → social service admin list/search
   - custom rooms → room map
   - queue → ranked queue counts
   - active matches + match detail → a live-match registry in the ranked service
3. **Durable operations store**: process-local Maps + optional JSON snapshot rewrite are not production-safe (loses everything on restart; corrupt snapshot silently treated as empty). Mirror the economy service: `node:sqlite` `DatabaseSync` (via `createRequire`) + `migrations/002_operations.sql` idempotent DDL + `{ dbPath? }` ctor defaulting `:memory:` under `NODE_ENV==='test'` and a file path in prod. Keep the service's public API identical so routes don't change.
4. **Feature flags are decorative until consumed**: GET/PUT flag records with no production caller is not a kill switch. Gate real routes with `operations.isFeatureEnabled(name)` (default `true` unless explicitly disabled): ranked queue + matchmaking, store purchase, redeem.
5. **Audit completeness**: cover every sensitive admin action — redeem-code creation, economy credit grants, flag changes — not just OperationsService's own mutations. Make `audit()` public and call it from the routes that mutate economy/ops state.

## Live-match registry pattern

- Record the match in `startMatch` AFTER `allocate()` succeeds (commit-step — a failed allocate must not register; matches the "commit after fallible external call" rule from earlier phases).
- The result/remake routes don't carry `match_id`, so close by mode + participant overlap: the busy-set guarantees a player is in at most one match, so any participant uniquely identifies the live match. No route contract change needed.
- Match detail returns what the API process actually knows (id/mode/server/players/startedAt/duration + per-player rating/tier). Deep match state (map/seed/god/loadout/energy/population) lives in the game-server process via the replay/postmatch pipeline — do NOT fabricate it in the API.

## Test patterns

- Restart persistence: two service instances on the same file DB; assert records + flags + audits survive.
- Route-level kill switch: PUT flag false → player queue returns `403 FEATURE_DISABLED` → re-enable → 200.
- Live monitor: two guest accounts → matchmaking → `/admin/live-matches` has 1 match, `/admin/matches/:id` returns players; `/admin/audit-logs` contains `feature.disable`/`feature.enable`/`redeem.code.create`.
- Pass `allocate` into `buildServer` in route tests that must create real matches — the default allocate fetches the game server and fails in tests.

## Resuming mid-phase work

The prior session's fresh-review verdict JSON is at `~/AppData/Local/hermes/cache/delegation/subagent-summary-<newest>.txt`. Read the newest file first — it lists exact blockers with evidence/line refs and an acceptance matrix, so you fix the right things instead of re-deriving the review. Also check `git status --short` before `git log` (phase work is usually uncommitted).

## Round-2 findings (post-fix re-review)

- **Phantom currency = silent no-op grant.** The compensation route accepted `essence`/`gold` while `Currency = 'coins' | 'shards'` and `grant()` only writes coins/shards — a 201 that credited nothing. Route allowed-value sets must equal the domain type's real values, and the route test must assert the wallet/inventory actually changed for every accepted currency AND that out-of-domain values get 400.
- **Presence must be derived, not hardcoded.** `status: 'online'` on every friend/admin record is a stub; derive online from real state (e.g. `SocialService.onlinePlayerIds()` = distinct session-map values) and expose it on admin player list + search.
- **Decorative UI controls are the same stub class.** A console search input that reloads the list without sending `?q=` looks wired but isn't; wire the query param and render real field names (e.g. `player_id`/`display_name`, not invented `id`/`name`).
- **When a re-review's delegated verdict never arrives** (HTTP 429 / max_iterations killed the final summary), grep the live transcript `~/AppData/Local/hermes/cache/delegation/live/<delegation_id>/task-0.log` for `think` lines and last tool results — blockers with file:line evidence are usually there. Fix them, re-run gates, re-dispatch; never treat a dead review as a pass.

## Verification

`pnpm exec prettier --write` on changed files BEFORE gates, then: typecheck, lint, format:check, `git diff --check`, build (api + gm-console), full `pnpm test` (counts: API + game-server + packages/testing). Docs drift: update docs/API.md Phase-N section and docs/DATABASE.md store notes when the contract changes.
