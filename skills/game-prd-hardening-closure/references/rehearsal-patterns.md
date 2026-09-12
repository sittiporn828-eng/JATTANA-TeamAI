# Rehearsal patterns

## Two-process API/game-server proof

Build both services, then start the executable entries (the game server uses `dist/main.js`; `dist/index.js` is the library export). Drive the real HTTP flow:

1. `POST /api/v1/accounts/guest` twice.
2. `POST /api/v1/ranked/queue` for both sessions.
3. `POST /api/v1/ranked/matchmaking`.
4. Assert `match_id` and `player_ids`.
5. `POST /api/v1/ranked/result` with `participant_ids`.
6. Assert the server release completed with the allocator's authoritative match ID.

Restart services between an authenticated load scenario and the standalone E2E rehearsal; the in-memory queue intentionally keeps live matches.

## Required release regression

Inject `release(serverUrl, matchId)` into `buildServer`, make the allocator return an ID that differs from `playerIds.join('-')`, submit a result, and assert the injected release receives the allocator ID. Keep allocation state until release succeeds.

## Browser event proof

A browser client must receive deployment-provided URL/match/player/session context, send a protocol-valid `player_ready` envelope on socket open, and trigger native notification only from the server's `match_found` envelope. A local custom event remains useful for permission testing but is not integration evidence.

## Generated artifact handling

Before cleanup, inspect untracked SQLite/temp files. Prefer repository ignore rules (`data/`, `.tmp-backup-*/`) over deletion; preserve runtime/user data unless deletion is explicitly authorized.

## Evidence boundary

A green unit suite proves implementation invariants, not PRD integration. Keep separate evidence for: unit/full gates, real multi-process E2E, authenticated load, backup/restore, alert delivery, browser handshake, and fresh fail-closed review.
