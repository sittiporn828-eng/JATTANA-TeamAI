# Phase closure pitfalls — learned on Godforge Phase 5 & 6 (2026-08)

Each pitfall below was caught by a fresh fail-closed review after green tests. Reviewers keep finding these; bake the fix in from the start.

## 1. Auth: bearer = player_id is NOT auth

Returning `player_id` in every response and then trusting `Authorization: Bearer <player_id>` lets anyone impersonate (player ids leak via search results). Fix pattern used:
- Mint opaque session tokens (`randomUUID`) at guest-create / login; `Map<token, playerId>`.
- `authenticate(token)` gate on every protected route → 401 on missing/forged.
- Mutations that take a target id (link-email, invite, kick, team assign) must authenticate the ACTOR, never trust ids in the body.
- Email link: reject if email already bound (`409 EMAIL_TAKEN`) — otherwise account takeover.
- Passwords: `scryptSync(password, salt, 32)` + per-user `randomUUID` salt + `timingSafeEqual`; never plain sha256.

## 2. External-service tickets must be redeemable

API "allocates" a game server by HTTP call, then hands the client `server_url` + `match_id`. If the game server rejects intents whose `match_id !==` its own startup `randomUUID()`, the ticket is unredeemable — a client following the API's answer cannot connect. Fix: the allocator returns the SERVER's real match id (or the server adopts the requested one). Test with a REAL server round-trip, not a mocked allocator — the mock hides exactly this bug.

## 3. Never set local state before an external call succeeds

`startRoom` set `room.match_id` before calling the allocator → on 503 the room was permanently `ALREADY_STARTED` with no server and no retry path. Fix: validate locally → call external → only then `confirmStart(roomId, actor, matchId)`. 503 must leave the resource retryable. Add the 503-then-retry test.

## 4. Reconnect: one player, one active socket

- Adapter must bind player identity to the socket, not re-read it from intents.
- `reconnect` through a new socket while the old socket is still closing → double-`reconnect()` calls hit `PLAYER_ALREADY_CONNECTED`. Route reconnect through the single intent handler; don't pre-call it in the adapter.
- Keep `activeConnections: Map<playerId, socket>`; stale socket `close` handler must not disconnect the replacement session (guard `activeConnections.get(playerId) === socket`).

## 5. Event-loop starvation: don't serialize the full world every tick

Broadcasting the complete world JSON at 20 TPS to every socket starves the event loop (WS clients stop responding, tests hang on `open`). Fix: full snapshot on join/reconnect; throttled projection (`server_tick % N === 0 ? world : undefined`) for periodic deltas.

## 6. Fastify 5 + strict TS

- `request.params` is `unknown` on string routes → declare route generics `scope.post<{ Params: { playerId: string } }>(...)`. Every route with params needs it.
- `exactOptionalPropertyTypes` rejects `prop = undefined` and `this.optional = options.optional` when optional is `string | undefined`: declare the field `string | undefined` (not `prop?: string`), or build objects with conditional spreads `...(x !== undefined ? { x } : {})`.
- Narrow before use: `if (!playerId) return undefined; const account = this.accounts.get(playerId);` — TS won't narrow `playerId` from an `account` check.

## 7. Test semantics: assert the behavior you mean

- Guest persistence: the FIRST guest call must already carry `client_id`, then re-create with the SAME `client_id` (a first call without id followed by a call with a new id is a different device, not a reconnect).
- Friend reject/block: the RECIPIENT rejects/blocks the SENDER (`rejector=bob, from=alice`), route path is `/friends/<target>/reject`.
- Party join requires an invite — a test that joins without inviting asserts the wrong thing; assert 403, then invite, then join 200.
- `exactOptionalPropertyTypes` and object-shape assertions: match response fields (e.g. `entry.from`, `entry.player_id`) to what the API actually returns.
- Persistence tests: persist `byClient` (client_id→player_id) too, not just accounts — otherwise guest progress "restarts" on reload.

## 8. Persistence baseline before real DB

In-memory Maps + JSON file snapshot (`node:fs`, zero deps) behind an optional `snapshotPath` satisfies "progress survives restart" for a phase; swap for the real DB when accounts matter. Snapshot must include every index map (byClient, byEmail, sessions), not just the entity store.

## 10. Ranked/matchmaking services (Phase 7)

- **Silent no-op when a participant is missing**: `reportResult` (and similar) that early-returns when a player isn't registered makes the route look fine while nothing happens. If only the actor was auto-created (`ensureRanked(actor)`), the OPPONENT may not exist yet → rating never changes, test fails at `> 1200`. Fix: `ensureRanked` BOTH winner and loser at the route boundary before calling the service.
- **Guard empty ids in auto-register helpers**: `ensureRanked('')` happily creates a player with an empty id if the guard is only `rating === 1200 && rank === 'UNRANKED'`. Add `if (!playerId) return;` first.
- **Hidden MMR ≠ visible rating**: `createPlayer(id, hiddenMmr)` seeds hidden MMR (matchmaking signal) but visible rating starts at the baseline (1200). Tests asserting `rating === hiddenMmr` are wrong; assert rating moves, and check hidden MMR separately if needed.
- **Placement count is part of the contract**: with 5 placement matches, a test playing 4 and expecting a tier fails correctly — play exactly the configured count (or read it from the service).
- **Matchmaking API tests need an allocate stub**: the default allocator fetches a real game server that isn't running in tests → `match_id: null`. Inject `buildServer({ allocate: async () => ({ server_url, match_id }) })` and assert the stub's id comes back; keep ONE real-server round-trip test (game-server `/allocate`) separate.
- **Elo sanity**: K=32 (double during placement), expected `1/(1+10^((rb-ra)/400))`; soft reset `1200 + (rating-1200)*0.5` + placements reset; remake penalizes leaver −K and leaves others untouched; `valid: false` skips the whole update.

## 9. Subagent review hygiene

- Review subagents work from your context; they may not actually open files. Verify via the delegation summary + live transcript (grep/read_file calls) before trusting "passed".
- Re-dispatch a fresh review after every edit round; an earlier pass does not cover later edits.
- Give reviewers the exact acceptance list and tell them what is OUT of scope (later phases) so they don't inflate blockers.
- If a review summary is truncated in-band, read the full file — the omitted middle often holds the real blockers.
- A delegation that returns "No reply: the turn was stopped because session storage could not be written" is a TRANSIENT infra failure, not a review verdict. Check disk space (`df -h`), then re-dispatch the same review — do not treat it as a result or as evidence the review found nothing.

## 11. Production wiring & authorization (Phase 7 final round)

Reviewers found these AFTER green tests passed — bake them in from the start for any queue/ladder/stateful service:

- **A feature only tests call is dead in production.** `tickQueue()` passed its unit test but nothing in `buildServer` ever called it → range never expanded in the real API. Wire background loops into the server entrypoint: `setInterval(() => ranked.tickQueue(30), 30_000).unref()` (`.unref()` so tests don't hang). Grep callers of every service method before declaring a flow done.
- **Cleanup must run before early-return guards.** `reportResult` had `busy.delete()` AFTER the `valid === false` return → an invalid match left both players stuck `busy` forever (queue refuses busy players; only a remake unstuck them, which also wrongly penalized −32). Move state cleanup above the guard (or `finally`). Same class as pitfall #3, but on the guard/error path.
- **Membership authorization on state-mutating endpoints.** Authentication ≠ authorization: any logged-in user could POST `/ranked/result` with arbitrary `winner_id`/`loser_id` (repeatable rating forgery, the PRD's "Rank Abuse Foundation"). Fix: require the named players to be in a live match — `isBusy(winner) && isBusy(loser)` → 403 `PLAYERS_NOT_IN_MATCH`; same for `remake` (`LEAVER_NOT_IN_MATCH`). Tests: forged result/remake → 403.
- **Global destructive ops need an admin gate.** `season/start` reset everyone's ladder when called by any authenticated player. Fix: env `RANKED_ADMIN_TOKEN` (+ `options.adminToken` for tests), check `token === adminToken` BEFORE session auth — otherwise a valid admin token hits `AUTHENTICATION_REQUIRED` (401) before the admin check (403). Test both 403 (user token) and 200 (admin token).
- **Restore queue state when allocation fails.** `startMatch` dequeues players before the external allocate call; on failure they were lost from the queue. Re-enqueue them in the catch.
- **Auto-register guard: use an existence check, not a value heuristic.** `ensureRanked` as `rating === 1200 && rank === 'UNRANKED'` re-created a mid-placement player sitting exactly at 1200, wiping placement progress. Use `if (!playerId) return; if (!hasPlayer(playerId)) create(...)`.
- **Order of authz checks in a route:** admin-token routes check admin first; participant routes check busy-membership before calling the service; always 401 for missing session, then 403 for authorization failures.

## 12. Late-phase hardening closure lessons

- **Release must reset the authoritative object, not just flags.** Changing `allocated`/lifecycle and an active ID while leaving the authoritative server's match ID, world, players, or sequence state intact is false cleanup. Use one domain-level reset/reconfigure path and test allocation → release → fresh snapshot.
- **Map validation must measure and gate.** Structural checks are not balance validation. Run a deterministic simulation window and report resource access, expansion speed, first-contact timing, population growth, and territory distribution; wire it into ranked admission or a fail-closed release gate.
- **Recovery endpoints need a durable caller and idempotent proof.** A recovery method alone is decorative; connect it to an operational/admin trigger or worker, use the same transactional uniqueness/ledger path as the original grant, and test retry/restart behavior.
- **Client notification test events are not production wiring.** A synthetic browser event proves only the local listener. Trace the actual API/WebSocket match-found event into the client boundary, then test listener cleanup and native permission/audio behavior.
- **Fresh review means CURRENT tree.** Any edit after a review invalidates that review; rerun gates and dispatch a new read-only review. A passing count (for example 133 tests) never substitutes for a zero-blocker acceptance matrix.
- Mark deliberate ceilings with `ponytail:` comments: process-local rate limits, local metrics, hardcoded seed lists, and heuristic abuse detection must not be reported as horizontally production-ready controls.
