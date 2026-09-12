# God Powers & Competitive Match Checklist

Use this reference when a deterministic simulation phase adds player-controlled powers and match win/loss logic. Values below are the validated Godforge v0.1 baseline pattern; re-read the current project config/docs before changing them.

## Vertical tracer

Prove the path with one real power before expanding the roster:

1. Create a match with unique players and civilizations.
2. Select a God and validate a 3–6 power loadout, budget ≤10, max one ultimate.
3. Start each player at 40/100 energy; regenerate 0.2/sec (1 per 5 sec), capped at 100.
4. Cast one data-driven power through the same server intent path production will use.
5. Deduct energy, set cooldown from the cast tick, and apply a deterministic world-state effect.
6. Reject an immediate recast; advance exact cooldown ticks; verify regen and recast.
7. Replay from the same seed/intents and compare serialized match states.

Passing this tracer means **vertical slice**, not phase completion.

## Server validation order

Fail closed in this order so rejected intents never mutate state:

1. Match status is `running`.
2. Player belongs to the match and controls the claimed civilization.
3. Power exists and is equipped for the selected God/loadout.
4. Power is not banned/disabled by the locked match config.
5. Energy is sufficient.
6. Cooldown has expired.
7. Target exists, is traversable/valid for the power, and respects ownership/range rules.
8. Opening protection permits that category/target (baseline: 300 sec, protected radius 8 tiles; destructive powers blocked).
9. Per-player power-intent rate ≤8/sec. Count every request at the transport/server guard, including rejected intents. If the gameplay API returns no updated state on rejection, do not store the authoritative limiter only inside immutable match state.
10. Ultimate unlock/slot/stack rules pass.

Every rejection needs a behavior test asserting no energy, cooldown, world, score, or event mutation. For APIs whose success result contains a huge world state, assert the small rejection projection (`ok`/`error`) or use `toMatchObject`; avoid equality assertions that print multi-megabyte world diffs on failure.

## Data and determinism rules

- Store costs, cooldowns, radii, durations, damage/effect caps, energy, loadout rules, score weights, and match duration in versioned game config—not `.env` or client constants.
- Snapshot balance/game/map/power config versions and seeds at match start for replay and audit.
- Keep power effects inside the existing authoritative simulation state/update path; do not create a parallel power-world model.
- Use integer ticks for cooldowns, warnings, durations, timers, and rate windows. Inputs and seeded tie-breakers must be explicit; never use uncontrolled randomness.
- For area effects, enumerate valid tiles deterministically and use stable ordering before applying caps.

## Match and victory closure

Match states: `waiting → countdown → running → capital_critical → finished`; also support `cancelled` and `invalid` exits.

Acceptance closure requires:

- God selection and loadout validation.
- Energy spend/regen/cap and cooldown boundaries.
- Supported power effects and safety caps.
- Match timer and power rejection outside `running`.
- Server score updated at the documented cadence (baseline UI cadence 5 sec).
- Weighted score on 0–10,000 scale: territory 20%, population 20%, cities 20%, economy 15%, technology 10%, military 10%, stability 5%.
- Conquest victory from authoritative capital/elimination state.
- Time-up score victory and documented tie-breakers.
- Capital-critical transition/recovery.
- Finished/rematch state that cannot accept gameplay intents.

## Full-roster closure

- A complete catalog is not a complete power system. Metadata plus a generic `ok` cast test proves dispatch only.
- If the user asks to finish the phase or prior status names remaining power effects, every power claimed as supported needs a real authoritative effect, documented duration/warning timing, target rules, safety caps, and its counter interaction where specified.
- Timed effects belong in the existing simulation update path and must survive replay; do not collapse a 60-second buff, DoT, hazard, or warning into an undocumented instant proxy.
- Put effect magnitudes, target caps, durations, damage caps, and other balance numbers in the versioned config. Do not hide a second balance table in a `switch` statement.
- `TBD_BY_TEST` is not permission to silently invent a production constant. Keep an explicit configurable baseline only when the source defines one; otherwise report the unresolved model/value as a completion blocker.
- Test representative semantics and boundaries, not just `Object.keys(catalog).length` or `{ ok: true }` for every ID.

## Review freshness and false-green prevention

- Bind independent reviews to the exact tree reviewed: record the commit plus dirty diff or hashes of every scoped file. After later edits, re-check findings against the current tree and require a fresh final review; a stale FAIL is not proof a fixed bug remains, and a stale PASS is not completion evidence.
- In workspaces whose tests import built package output, build/typecheck producer packages before targeted tests. Test-only commands can consume stale `dist` artifacts and report green while current TypeScript does not compile.
- Catalog smoke tests must construct valid archetype/loadout setups through the public match-creation path. Never forge a one-power loadout by mutating validated match state.
- “Data-driven” means effect magnitudes, caps, timing, warning/expiration data, and target rules are versioned configuration consumed by the authoritative update path—not metadata beside hardcoded behavior.
- Temporary effects must live in authoritative tick-based state and expire deterministically. Do not represent buffs/debuffs by mutating derived scores that the next simulation update recomputes.
- Keep warning lead-time separate from effect duration: for a 3-second warning plus 12-second hazard, preserve `duration=12` in the catalog and expire at `cast + warning + duration`; test both boundaries.
- Separate authoritative score cadence from presentation cadence. A 5-second UI refresh must not silently become a 5-second server score cache when the balance rules require per-tick state.

## Verification gates

- RED→GREEN per new vertical behavior; do not batch imagined tests for every power. If production code already exists, add a characterization/regression test—do not delete working code merely to manufacture a RED run.
- After the final code or config edit, rerun the canonical full suite and latest-tree independent review. Older green runs and stale reviews are not completion evidence.
- Negative validation matrix with no-mutation assertions.
- Energy/cooldown boundary tests at `readyTick-1`, `readyTick`, zero energy, and cap.
- Opening-protection and ultimate-unlock boundaries.
- Score component normalization, weight sum, time-up winner, and tie-break tests.
- Conquest ownership consistency across tiles, territory, settlements, buildings, resources, units/population, and events.
- Multi-seed deterministic replay, finite/non-negative state, full lint/typecheck/test/build/format/diff checks.
- Independent fail-closed review of the latest tree after the final edit.
