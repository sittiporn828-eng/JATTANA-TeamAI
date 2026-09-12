# Godforge Phase 7 & 8 — Ranked modes and team systems (verified 2026-08-10)

Session-verified detail for `RankedService` (Phase 7) and TeamSystem/FFA/DraftRules (Phase 8).
The recurring blocker class was **production wiring + route authz**, not domain math — all four
Phase 7 blockers were invisible to green unit tests.

## Phase 7 acceptance (PRD) — all passed after 2 review rounds

1. Queue finds close-MMR players (hiddenMmr range ±100 base, +25 per 30s, cap ±500)
2. Rating up/down correct (Elo: `E = 1/(1+10^((rb-ra)/400))`, zero-sum `±round(K*(1-E))`, K=32, ×2 during placement)
3. Rank modes separate (independent ModeState per 1v1/2v2/ffa)
4. Placement: 5 matches/mode → tier BRONZE..MASTER; UNRANKED until done
5. Soft reset: `rating = 1200 + (rating-1200)*0.5`, placements reset, season counter++
6. Invalid match (`valid:false`) changes nothing
7. Remake: leaver −32 + games+1; other players untouched

Hidden MMR blends 90/10 with visible rating after each result.

## Phase 7 blockers the first fail-closed review found (all fixed)

1. **Dead-in-production wiring**: `tickQueue()` had tests but no caller — range never expanded
   in the running API. Fix: `setInterval(() => ranked.tickQueue(30), 30_000).unref()` in
   `buildServer`. General rule: grep callers of every domain function before claiming complete.
2. **Busy-set leak on invalid match**: `reportResult` returned on `valid === false` before
   `busy.delete`, so a server-error match bricked both players' queues forever. Fix: clear
   busy for winner/loser BEFORE the `valid === false` early return. Rule: every early-return
   path must run its state cleanup.
3. **Rating forgery via result/remake routes**: any authenticated user could POST arbitrary
   `winner_id/loser_id/leaver_id` (repeatable, unverified). Fix: routes require
   `ranked.isBusy(winner) && isBusy(loser)` → 403 `PLAYERS_NOT_IN_MATCH`, and
   `isBusy(leaver)` → 403 `LEAVER_NOT_IN_MATCH`; check BEFORE `ensureRanked` (which would
   mint fresh never-busy players). Rule: result/remake-style routes need match-membership authz.
4. **Global season wipe by any player**: `/ranked/season/start` needs an admin gate.
   Fix: `adminToken = options.adminToken ?? env.RANKED_ADMIN_TOKEN` (shared env schema),
   route checks `!adminToken || token !== adminToken` → 403 `ADMIN_REQUIRED` as the FIRST
   line, before session auth (otherwise the admin token itself gets 401'd by session auth).

Extra fixes from the same review: `startMatch` re-enqueues players on allocate failure
(findMatch already dequeued them); `ensureRanked` must use `hasPlayer(playerId)` — the
heuristic `rating === 1200 && rank === 'UNRANKED'` silently wipes placement progress of an
existing player sitting exactly at 1200.

## Phase 8 acceptance (PRD) — 2v2, FFA, Pick/Ban

- 2v2: 4 players → blue[2] / red[2] (formation + global assignment map), shared team score
  = sum of member scores, team ban: blue 2 + red 2 with per-team `legalFor()`, party rank
  spread check.
- FFA: winner = last standing, else top score among ACTIVE civs only; `activeScores()`
  filters eliminated out of ranking.
- Pick/ban rank rules (server-issued `draftConfigFor(tier)`):
  BRONZE/SILVER `none`, GOLD/PLATINUM `light`, DIAMOND `one-per-side`, MASTER+ `two-per-side`.
- RankedService.findMatch extended: `2v2`/`ffa` need 4 players within range (1v1 needs 2).

Design smell flagged by reviewer (leave until a real phase needs it): FfaRules.eliminated()
reads a module-level hardcoded set for a unit test — real elimination must come from match
state, not a global constant.

## Phase 8 blockers the first fail-closed review found (all fixed)

1. **2v2/FFA result flow was still 1v1-shaped — busy leak for teammates**: `findMatch` marks
   all 4 busy, but `reportResult` accepted exactly one winner+loser and cleared only 2 busy;
   the other 2 teammates stayed busy forever (only escape was `/remake`, which docked −32).
   Fix: `MatchResult` gains `participantIds?: readonly string[]` (default `[winnerId, loserId]`);
   `reportResult` busy-deletes every participant BEFORE the `valid === false` early return;
   `/ranked/result` accepts `participant_ids`, requires `participantIds.length >= 2` and every
   id `isBusy` (403 `PLAYERS_NOT_IN_MATCH` otherwise). Rule: for N-player matches the result
   payload must name every participant, not just winner/loser.
2. **Team shared objective had zero production callers**: `teamScore()` was unit-tested only.
   Fix: `POST /modes/teams` accepts `{player_ids, scores}` and returns per-team `team_scores`.
   `teamScore` signature changed to take explicit `members` (dropped the global assignment map).
3. **Team ban had zero production callers**: `resolveBans()` unit-tested only; no endpoint.
   Fix: new `POST /modes/teams/ban {blue_bans, red_bans}` returns `banned_powers` +
   per-side counts. General rule: a domain helper without a route/loop caller is dead in
   production regardless of test coverage.
4. **Test fixture baked into production state**: `ELIMINATED_CIVILIZATIONS = ['civ-c','civ-d']`
   was a module-level constant the unit test relied on — the reviewer's earlier "design smell,
   leave it" note became a blocker once the route trusted the client's pre-filtered `active`
   list. Fix: deleted the constant and `eliminated()` entirely; the `/modes/ffa/winner` route
   now calls `FfaRules.activeScores(scores, eliminated)` then `winner(active, activeScores)` —
   the server derives elimination, never trusts client filtering.
5. **No roster in matchmaking response**: client got `{match_id, server_url}` only, so
   `/modes/teams` was undrivable. Fix: response spreads `player_ids: match.playerIds`.
6. **Duplicate player ids accepted**: 4× same id formed `[x,x]` teams. Fix:
   `/modes/teams` rejects `new Set(playerIds).size !== 4` → 400 `REQUIRES_FOUR_DISTINCT_PLAYERS`.
7. **Global `TEAM_ASSIGNMENT` map removed**: module-level mutable map keyed by player id
   clobbered across matches and mutated global state from any caller; `formation()` is now a
   pure function, `teamOf` deleted, `teamScore` takes explicit members.

Verified: 77 tests (39 API + 4 game-server + 34 regression), gates green after the round.

## TypeScript-strict pitfalls (bit us repeatedly, all in this repo's tsconfig)

- `exactOptionalPropertyTypes: true` rejects `this.x = options.x` when `x?: string` and
  options.x is `string | undefined`. Fix: declare `private readonly x: string | undefined;`
  (non-optional), or `const p = this.x; if (p) ...` locally (also fixes `this.x!` smells).
- `noUncheckedIndexedAccess: true` makes `entries[i]` destructuring fail
  (`Type '[string, number] | undefined' must have a Symbol.iterator`). Fix: grab the entry
  into a local, `if (!entry) continue;` then destructure.
- Fastify 5: `request.params` on a plain string route is `unknown` — type routes as
  `scope.post<{ Params: { playerId: string } }>(...)`; `request.body` can be null, so
  `(request.body ?? {}) as ...`.
- fastify `app.inject` + `payload` object auto-sets JSON content-type; body is parsed.
- Test traps: reject/block semantics are receiver-acts-on-sender (bob rejects alice's
  request), party join requires an invite, guest progress requires reusing the same
  `client_id`. Get the actor semantics right in tests before blaming implementation.

## Godforge ops (this repo)

- PRD file: `read_file`/`search_files` fail on `docs/Godforge_Master_PRD_TDD_Phase_Based_v2.2.md`
  (returns ~200 chars or no matches). Read phase scope with
  `python -c "s=Path('docs/...md').read_text(encoding='utf-8'); i=s.find('# PHASE 8'); j=s.find('# PHASE 9'); print(s[i:j])"`.
- Full gates before review: `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && git diff --check`.
- Push: `GIT_SSH_COMMAND='ssh -i ~/.ssh/id_ed25519_github_meekamrai -o IdentitiesOnly=yes' git push origin main`
  (sittiporn828/Godforge uses the meekamrai key; `work1` authenticates but has no repo rights).
  Verify with `git ls-remote` + local HEAD equality after push.
- Never commit/push until the user explicitly asks, and never declare a phase complete while
  the fresh fail-closed review is pending — both were enforced repeatedly this session.
- Delegated review subagent may fail with "session storage could not be written" — usually a
  transient (disk had 109G free, cache 17M); re-dispatch rather than treating it as a verdict.
