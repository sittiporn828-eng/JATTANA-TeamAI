# Godforge Phase 6 — Social API layer (Fastify 5, in-memory domain)

Verified working patterns from the Phase 6 implementation (2026-08-10). Tests: 9 API
behavior tests via `app.inject()`; full suite 46 tests green.

## Structure that made it testable

- `apps/api/src/social.ts` — `SocialService` class, pure in-memory Maps, no Fastify
  imports. Returns result unions (`'created' | 'duplicate' | 'not_found'`,
  `Room | undefined | 'not_ready'`) so routes map to status codes without domain checks.
- `apps/api/src/server.ts` — `buildServer(options: { social?: SocialService })` returns
  the Fastify app; tests inject a fresh service per test. No `listen()` in this file.
- `apps/api/src/index.ts` — re-exports `buildServer`; only listens when run as the
  process entrypoint (same side-effect-free pattern as game-server `main.ts`).
- Tests: `app.inject({ method, url, payload, headers })` — full route stack, no port.

## Fastify 5 gotchas (both hit during this phase)

1. `request.params` is typed `unknown` unless the route declares generics:
   ```ts
   scope.post<{ Params: { playerId: string } }>('/api/v1/friends/:playerId/request', ...)
   // and for two params:
   scope.delete<{ Params: { partyId: string; playerId: string } }>('/api/v1/party/:partyId/members/:playerId', ...)
   ```
   Missing generics → TS18046. `request.body` stays an `as`-cast (no generic needed).

2. `exactOptionalPropertyTypes: true` in the repo tsconfig rejects
   `{ world_modifier: undefined }` when building an optional-field object. Conditional
   spread instead:
   ```ts
   {
     ...(body.world_modifier !== undefined ? { world_modifier: String(body.world_modifier) } : {}),
     ...(body.password !== undefined ? { password: String(body.password) } : {}),
   }
   ```

## Actor-semantics test traps (the RED failures were test bugs, not code bugs)

- **Guest identity**: create guest WITH `client_id`, reconnect with the SAME
  `client_id` → same `player_id`. First call without `client_id` then a second call
  with one creates two accounts (by design — a new device).
- **Pending friend requests**: response entries expose `from`/`to`, NOT `player_id`.
  Assert `entry.from === senderId`.
- **Reject/block direction**: the RECEIVER acts on the SENDER.
  `POST /api/v1/friends/:senderId/reject` called by the receiver; the sender is the
  route param. Flipping actor/target yields 404 because the request key
  (`from:to`) is looked up the wrong way.
- **Party join requires a prior invite** (PRD: "Party Invite ทำงาน"). Test 2 must
  invite before join, or the first join is 403. Membership check (`'duplicate'` →
  409) must run BEFORE the invite check in the service.

## Room start → game server allocation ticket

`POST /api/v1/rooms/:roomId/start` (host only) → 201:
```json
{ "match_id": "<uuid>", "server_url": "<GAME_SERVER_WS_URL>", "seed": 1, "config_version": "v0.1.0" }
```
- Blocked with 409 `NOT_ALL_READY` until every member `ready: true` and ≥2 members.
- Added `GAME_SERVER_WS_URL` + `GAME_SERVER_SEED` to `packages/shared` envSchema
  (zod defaults keep local runs deterministic).
- `GAME_CONFIG_VERSION` default changed `0.1.0` → `v0.1.0` so the version string
  matches the `v\d+` protocol convention.

## Post fail-closed review hardening (second pass)

The first review rejected the happy-path version. These blocker fixes ship WITH the phase:

- **Auth is not a player_id**: bearer credentials must be opaque session tokens (`randomUUID` per guest-create/login, token -> player_id map). Never accept the raw player_id as a credential — it leaks via search/friend responses and makes impersonation trivial. All friends/party/rooms routes 401 on missing/forged tokens.
- **link-email must authenticate** as the account being linked and return 409 `EMAIL_TAKEN` when the email already maps elsewhere; otherwise an attacker links their email to a victim player_id (account takeover).
- **Passwords**: `node:crypto` scrypt + per-account random salt + `timingSafeEqual`. Unsalted single-pass sha256 is a blocker.
- **Persistence**: snapshot accounts/clientIds/sessions/friendships/pending/blocked to a JSON file on mutation (injectable `snapshotPath`); test restart recovery. In-memory-only passes tests but fails the "guest progress survives" acceptance.
- **Room password**: join compares the password (403 `WRONG_PASSWORD`); a deny-all branch makes passworded rooms unjoinable — dead code.
- **Allocation must be real**: game server exposes `POST /allocate` (validates matchId/seed/configVersion, returns its own ws server_url); the API calls it with map/duration/power-bans and 503s on failure; start-once guard (409 `ALREADY_STARTED`). Inject a fake `allocate` in API tests to assert the payload; test the real route in the game-server suite.
- **Lifecycle hygiene**: kick/leave revokes invites (rejoin after kick = 403 — the original test asserting 200 enshrined the bug), leave transfers host/leader, member caps (8), `Number.isInteger` for team (1–4) and duration (NaN passes `< 1` / `> 4` comparisons).
- **search() must not leak emails** — return a stripped account projection.

## Housekeeping

- After running dev servers, `git status --porcelain` may show stray untracked
  artifacts (a `ตัวอย่าง/*.apk` dir appeared at repo root and was not ignored).
  Delete and re-check before commit; don't let binaries leak into the phase commit.
- Push on this machine: `origin` is `sittiporn828/Godforge`; use the meekamrai SSH
  key (`GIT_SSH_COMMAND='ssh -i ~/.ssh/id_ed25519_github_meekamrai -o IdentitiesOnly=yes'`).
  `work1` key authenticates but has no access to this repo.
