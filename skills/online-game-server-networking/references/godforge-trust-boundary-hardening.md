# Godforge trust-boundary hardening (ranked + game-server auth)

Session: bug audit found 4 confirmed trust-boundary bugs despite green tests. All four share one root cause:
the game-server never bound authorized players or surfaced its authoritative outcome, so the API trusted the
client. Fixes are the canonical example of the SKILL.md trust-boundary rules.

## Repo layout
- `apps/game-server/src/index.ts` — HTTP control routes (`/allocate`, `/release`, `/metrics`, `/result`) + WS handler.
- `apps/game-server/src/server.ts` — `GameServer` class (`join`, `handleIntent`, `tick`, `snapshot`, `outcome`).
- `apps/api/src/server.ts` — ranked result/remake routes, `buildServer` injectables (`allocate`, `release`, `resolveOutcome`).
- `apps/api/src/ranked.ts` — `reportResult`, `beginSettlement`, `liveMatchForPlayers`.
- `packages/simulation/src/index.ts` — authoritative `CompetitiveMatch` already computes `winnerCivilizationId`/`victoryType`.

## Bug 1 (critical) — ranked result was client-authoritative
- `POST /api/v1/ranked/result` accepted `winner_id`/`loser_id`/`valid` from the client and only checked match
  membership before settling rating/progression.
- Fix: game-server exposes `outcome()` → `{status, winnerPlayerId, victoryType}` mapping `winnerCivilizationId`
  → playerId via the `players` map; surfaced over authenticated `GET /result`.
- API: new injectable `resolveOutcome(serverUrl, matchId)` (mirrors `allocate`/`release` injection pattern,
  default fetches `{serverUrl}/result`). Result route now resolves the server outcome; rejects 409
  `MATCH_NOT_FINISHED` if not finished, 503 `OUTCOME_UNAVAILABLE` if unreachable, 409 `OUTCOME_MISMATCH` if
  winner not in participants. `valid` field removed from the body — winner/loser derived from server.
- GameServer `outcome()` returns `not_started` when no match, `running` when no `winnerCivilizationId` yet.

## Bug 2 (high) — play before allocate / after release
- WS handler checked only `input.match_id === activeMatchId`, never `allocated`/`lifecycle`. Release resets
  `activeMatchId` to a fresh startup UUID.
- Fix: one guard at the top of the WS message handler: `if (!allocated || lifecycle !== 'running') →
  SERVER_NOT_RUNNING`. Single choke point — all intents route through it.

## Bug 3 (high) — initial identity not bound to auth
- `player_ready` accepted only literal `player-a`/`player-b`; session token issued AFTER join, so any client
  knowing the active match id could claim a slot.
- Fix: allocate request now carries `players: string[]` (exactly 2, unique, `[A-Za-z0-9_-]{1,64}`). Game-server
  stores `authorizedPlayers` set; `player_ready` rejected unless the id is in it → `PLAYER_NOT_AUTHORIZED`.
  The literal slot check was removed — the server now binds the real account ids the API passes
  (`match.playerIds` for ranked, `room.members.keys()` for rooms).
- `allocate` injectable type in API gained `players: readonly string[]`.

## Bug 4 (medium) — unbounded progress overwrite
- `SocialService.updateProgress` used `Math.max` (forward-only) with no ceiling → client could jump
  `tutorial_step`/`practice_matches` to any value.
- Fix: cap to `current + 1` per call, so a client can only advance one step at a time.

## Test updates required by the new contract
- Game-server tests that allocate must now include `players`. New test: reject play before allocation
  (`SERVER_NOT_RUNNING`), reject unauthorized player (`PLAYER_NOT_AUTHORIZED`), missing `players` → 400,
  `GET /result` returns `{status:'not_started', authoritative:true}`.
- Ranked-api tests: inject `resolveOutcome` stubs (finished with a winner) since default now fetches a real
  game-server. Add test asserting a non-finished outcome → 409 `MATCH_NOT_FINISHED`.

## Pitfall hit this session (verification discipline)
Bulk `patch` edits across several test files mangled `authorization: Bearer ${token}` into
`authorization: *** ${token}` (tool redaction collapsed the literal). Session ended with a syntax error in
`ranked-api.test.ts` still pending. Lesson: after bulk edits, grep for `***` / re-read auth-header lines and
run the affected suite before declaring done; never end a turn with a known syntax error.
NOTE: `***` in tool OUTPUT (read_file / cat) is Hermes redacting `Bearer ${...}` in DISPLAY — grep the file
directly (`grep -c '\*\*\*'`) to check the real bytes, not the rendered output.

## Pitfall hit this session (state-machine regression — caught by fail-closed review)
Adding fail-closed error paths to a flow that ALREADY acquired a state lock wedges the flow if the early
returns don't restore the lock. In the ranked result route, `beginSettlement` flips the live match to
status `'settling'`; the new 409 `MATCH_NOT_FINISHED` / `OUTCOME_MISMATCH` paths returned WITHOUT
`restoreSettlement`, so the match was permanently stuck 'settling' (could never settle, players stayed busy,
allocation leaked). Fix: call `restoreSettlement(mode, participantIds)` on EVERY early-return rejection
path, not just the success/`OUTCOME_UNAVAILABLE`/release-failure paths.
General rule: for any `begin*`/`acquire`/lock operation, enumerate every `return` after it and make each
one restore the state before responding. Add a test that proves a rejected match can be settled on a
retry once the condition clears (stub resolves `running` first, then `finished`) — a wedged-match test
would return 403 `PLAYERS_NOT_IN_MATCH`/`MATCH_MEMBERSHIP_REQUIRED` on retry instead of 200.

## Whole-flow rehearsal findings

Unit tests with an injected `resolveOutcome` stub can hide drift in the real HTTP seam. The game-server
wire response used camelCase (`winnerPlayerId`, `victoryType`) while the API consumer expected snake_case
(`winner_player_id`, `victory_type`). Add one integration assertion against the actual `/result` JSON and
parse exactly the producer's public contract. Keep domain and availability failures distinct: a reachable
`status: running` result must become retryable 409 `MATCH_NOT_FINISHED`; only transport, authentication,
or malformed-response failure should become 503 `OUTCOME_UNAVAILABLE`.

Allocation schemas can also accept decorative configuration. Trace every accepted room field through to
authoritative match creation and assert behavior, especially `match_duration_seconds`, `power_bans`, map,
and config version. Validate unsupported bans/durations before setting lifecycle to reserved/running.
A response echo or stored field is not proof the simulation consumed it.

For bounded live rehearsal, never restore the old client-authoritative result shortcut just to make E2E
fast. Drive two actual WebSocket clients through allocation, ready, authoritative power propagation,
premature-result rejection, and authenticated remake/release cleanup. Close sockets in `finally`, or a
failed assertion leaves the Node process and allocation alive and contaminates the next run. Restart both
API and game-server before re-running a stateful rehearsal after manual control-plane intervention; API
ranked state and game-server allocation state otherwise diverge.
