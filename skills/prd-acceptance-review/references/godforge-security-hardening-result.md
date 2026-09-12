# Godforge security-hardening diff review — result-authoritative / lifecycle / binding

Review of an uncommitted working-tree hardening diff (`HEAD c99a118`, main) in the Godforge monorepo
(`C:/Users/Acer/Desktop/JATANA GROUP/Godforge`). Files: `apps/game-server/src/{index,server}.ts`,
`apps/api/src/{server,social}.ts`, plus tests. Goal: confirm 4 claimed trust-boundary fixes at root
cause and catch regressions. Verdict: all 4 root-cause fixes correct; one diff-INTRODUCED regression +
one PRE-EXISTING gap. Use `git show HEAD:<file>` to separate the two.

## The 4 fixes (each correct at root cause)

1. **CRITICAL ranked result client-authoritative.** Before: client sent `winner_id`/`loser_id`/`valid`
   and the API trusted them. After: game-server exposes `GameServer.outcome()` (reads
   `match.winnerCivilizationId`, set only by server simulation — elimination `living[0]` or
   `scoreWinner`; no intent can write it) + `GET /result` (token-gated
   `GAME_SERVER_RESULT_TOKEN ?? GAME_SERVER_ALLOCATE_TOKEN`, fail-closed 503 when unset outside
   `NODE_ENV=test`). API `resolveOutcome` GETs `/result` and hard-requires `status==='finished'`;
   result route derives winner from server outcome, ignores client fields, drops the `valid` flag
   (always true). Confirmed no WS/intent path sets the winner → unforgeable.
2. **HIGH WS accepted play when unallocated/released.** Guard `if (!allocated || lifecycle !== 'running')`
   placed at TOP of WS message handler before type dispatch → covers player_ready/reconnect/use_power/
   ping/surrender/camera_interest. Sits after rate-limit + parseIntent (parse is side-effect-free).
3. **HIGH initial player identity not bound to auth.** `allocate` now requires exactly 2 unique
   regex-valid player ids; game-server stores `authorizedPlayers`, clears on release; `player_ready`
   rejects ids outside it (`PLAYER_NOT_AUTHORIZED`). API passes server-controlled ids
   (ranked `match.playerIds`; room `[...members.keys()]`). Both `allocate` call sites updated.
4. **MEDIUM progress arbitrary overwrite.** `updateProgress` is the only writer of
   `tutorial_step`/`practice_matches`; each field = `min(current+1, max(current, requested))` →
   non-decreasing, no backwards, max +1 per call. Grind-by-repeated-calls is legitimate progression.

## Holes found

- **REGRESSION (introduced) — wedged settlement.** Result route: `beginSettlement` sets match
  `running → settling`, then the diff's NEW validation returns bail WITHOUT `restoreSettlement`:
  `if (outcome.status !== 'finished' || !winnerPlayerId) return 409 MATCH_NOT_FINISHED` and
  `OUTCOME_MISMATCH`. Since `beginSettlement` requires `status==='running'`, a participant POSTing
  `/result` while the game still runs permanently wedges the match → never settles, players stay
  busy, allocation leaks. Success path cleans up (`reportResult → closeLiveMatch`). Fix: restore on
  the 409 paths too (or confirm finished before claiming).
- **PRE-EXISTING gap — binding bypass via sibling intents.** `authorizedPlayers` gates only the
  `player_ready` branch. An unbound second socket can send `use_power`/`surrender`/`ping` with a
  joined authorized player's id; `handleIntent`/`usePower` validate NO session token (only `reconnect`
  does). Impersonation of an authorized player (spend their powers / surrender them). Identical
  structure at HEAD → not introduced here, but the "binding can't be bypassed" claim is incomplete.
  Fix direction: bind socket→playerId (reject intents whose playerId isn't the bound connection) or
  require the session token on all non-idempotent intents.

## Verification method worth reusing

- Read full source, not just the diff — trace every message type into the simulation to prove the
  authoritative field can't be written by a client.
- `git show HEAD:<file>` on touched paths to classify pre-existing vs diff-introduced before
  labeling anything a blocker.
- For a claim-before-await flow, enumerate EVERY return path added by the diff and check the claim
  is rolled back on each (the diff's own new 4xx branches are the ones that slip).
- Count that every `allocate()` call site now passes the required `players`.
