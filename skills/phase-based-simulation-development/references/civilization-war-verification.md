# Civilization & War Phase Verification

Use this checklist when a deterministic simulation phase introduces civilizations, territorial conflict, capture, or elimination. Replace example ranges with the product's source-of-truth values.

## Requirement matrix

| Class | Questions | Evidence |
|---|---|---|
| Hard acceptance | Can entities form civilizations, grow stages, expand territory, recruit armies, detect rivals, fight, capture, attack capitals, and eliminate? | One behavior test per transition |
| Explicit prohibitions | Is war forbidden from being a clock-only trigger? Are uncontrolled RNG and parallel state models forbidden? | Negative tests |
| Quantitative targets | Are population, city count, army share, and first-conflict timing given as ranges? | Checkpoint probes over representative seeds |
| Architecture | Does the implementation remain in the canonical server-compatible state/update path? | Dependency and caller inspection |

## Competitive fairness

- Starting resources, technology, military power, stability, and other competitive stats are symmetric unless the specification says otherwise.
- Seeded variation belongs in maps, AI choices, and tie-breaking—not hidden starting power bonuses.
- Verify different seeds can choose different actors while identical seeds replay byte-for-byte.

## Conflict-state boundary

- If the phase supports only 1v1, validate and reject larger match sizes at creation.
- If the API accepts 3+ civilizations, wars, captures, and critical timers must be keyed by conflict/target; global `find(kind)` and `hasEvent(kind)` checks are insufficient.
- Never advertise a broader match mode than the state machine can process correctly.

## Timed transition checks

For each timer, assert elapsed ticks from the event that starts it—not from world time zero:

1. Rival detected from spatial/threat state.
2. War declared from scored conditions, with a negative balanced/no-reason case. Keep normal/default aggression in that negative test and neutralize the actual causes (military/economy advantage, resource pressure, instability); setting aggression to zero can mask a clock-only declaration bug.
3. Defenders present at the capture target: capture progress stops. A global army count is not equivalent in a spatial simulation; if 1v1 deliberately uses aggregate presence, document that ceiling and test it explicitly.
4. Defenders absent: city hold reaches its configured duration.
5. Capital hold reaches its configured duration.
6. Critical window reaches its configured duration.
7. Defender recovery inside the window clears critical state.

Test off-by-one boundaries: one tick before, exact threshold, one tick after.

## Atomic conquest invariant

After city capture, every relevant projection agrees on the new owner:

- settlement/city state
- city-center or attached buildings
- tile/region ownership
- civilization city and territory indexes

After civilization elimination, the defeated owner must not remain active in:

- owned tiles/territory
- settlements/buildings
- resource nodes
- units/humans/population
- city lists, armies, enemies, or pending capture state

Either transfer or explicitly neutralize each projection in one state transition. Probe for stale owner IDs across the entire state.

## Balance checkpoints

At every specified milestone (for example 5/10/15/20/30 minutes), record per civilization:

- population
- settlements/cities
- stockpiles and finite/non-negative values
- territory
- army share
- technology/stability
- conflict event timing

Treat playtest ranges as calibration gates when the phase claims balance completeness. Do not infer success from the final checkpoint alone.

## Completion closure

1. Focused RED observed for each new behavior.
2. Focused GREEN observed.
3. Full lint/typecheck/test/build/format/diff checks pass.
4. Multi-seed deterministic and invariant probes pass.
5. Independent reviewer checks the latest diff.
6. Reviewer findings are fixed and re-reviewed.
7. No edits occur after the final suite; if they do, repeat the affected checks and full suite.

If the session/tool budget ends after the last edit but before step 7, report **unverified**, not complete.
