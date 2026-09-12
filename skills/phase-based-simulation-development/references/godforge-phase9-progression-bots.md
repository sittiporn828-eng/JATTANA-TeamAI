# Godforge Phase 9 — Tutorial, Bots, Progression, Missions & Achievements

## Scope (PRD Phase 9)

Tutorial flow (10 ordered steps), bot system (easy/normal/hard/expert), account XP/level
with content unlocks, per-god mastery, daily/weekly missions, achievements.
Acceptance: tutorial completes; bot uses core powers; XP recorded; level unlocks content;
mastery per god; daily/weekly reset; achievement triggers.

## Implemented shapes (verified passing)

- `apps/api/src/progression.ts` — `ProgressService`:
  - `TUTORIAL_STEPS` const array of 10 literals; `completeStep` only accepts the exact
    next step (out-of-order → false → route 409 `STEP_OUT_OF_ORDER`).
  - XP → level: `Math.floor(xp / 100) + 1`; `LEVEL_UNLOCKS` map level→content
    (`ranked`@1, `custom_rooms`@5, `god_mastery`@10).
  - God mastery: `godXp: Record<godId, xp>` per player, level = floor/100 + 1.
  - Missions: `MISSIONS` map missionId → `{cadence: daily|weekly, goal}`;
    `resetDaily()`/`resetWeekly()` delete only that cadence's entries.
  - Achievements: `Set` per player, `trigger()` idempotent.
- `apps/api/src/bot.ts` — `BotService` is an object of static methods (NOT a class):
  - `BOT_DIFFICULTIES = ['easy','normal','hard','expert']`, `TIER` map for difficulty ordering.
  - `decide(bot, state)` order: energy < threshold → `save_energy`; threat high + enemy
    cities → `defend`; expert + 3+ enemy cities → `attack`; else first off-cooldown power
    → `use_power`. `ENERGY_SAVE_THRESHOLD` per difficulty (expert saves at lowest).
  - `isBot: true` + `impersonatesHuman()` guard — bots must never enter ranked as humans.
- Routes (all auth-gated via `social.authenticate`): GET `/progression/tutorial`,
  POST `/progression/tutorial/step`, POST `/progression/xp`, GET `/progression/status`,
  POST `/progression/mission`, POST `/progression/reset {cadence}`,
  POST `/bots {difficulty, player_id}`, POST `/bots/decide`.
- Test layout that worked: domain tests (`progression.test.ts`, `bot.test.ts`) first,
  then HTTP tests (`progression-api.test.ts`) via `app.inject()`.

## Pitfalls hit this round (TypeScript strict)

- Union action types: `expect(action.powerId)` on a `use_power | save_energy | ...` union
  fails typecheck — narrow first: `if (action.type === 'use_power') expect(...powerId)`.
- Literal-union params: passing a non-member string (e.g. `'step-5'`) needs
  `as TutorialStep` cast in the negative test.
- Injectable static-object services (BotService) must be typed in BuildServerOptions as
  `bots?: typeof BotService`, not `bots?: BotService` — TS2749 otherwise.
- Adding a route that reads a new service field (`progress.totalXp`) requires adding the
  method to the service before the route typechecks.

## Exploit caution (unresolved at write time)

`POST /progression/xp` accepts an unbounded self-awarded `amount` from any authenticated
player — a level-grind/rank-unlock vector. Expect the fail-closed review to flag it;
the fix is a per-source XP ledger (server awards XP from match/tutorial/mission events,
client never posts raw XP) or a strict cap. Same class: `POST /progression/mission`
accepts arbitrary progress increments.

## Environment quirks (Godforge)

- PRD reading: `read_file` and `search_files` fail on the PRD markdown (large file /
  non-UTF8 bytes, sometimes read as binary). Use
  `python -c "from pathlib import Path; s=Path('docs/Godforge_Master_PRD_TDD_Phase_Based_v2.2.md').read_text(encoding='utf-8'); i=s.find('# PHASE N'); j=s.find('# PHASE N+1'); print(s[i:j])"`
  — this worked every phase.
- Delegation results arrive truncated in the batch-complete message; read the full file:
  `C:\Users\Acer\AppData\Local\hermes\cache\delegation\subagent-summary-<ts>.txt`.
- A subagent finishing with "session storage could not be written" is a transient
  failure (disk was 52% full, cache 17M) — re-dispatch the review, don't treat it as a verdict.
- The `patch` tool's lint output frequently shows `TS6053: File ... not found` because
  tsc runs with a path resolved from a different cwd. Ignore it when the package's own
  `pnpm --filter X build` passes; it is pre-existing noise, not a real error.
