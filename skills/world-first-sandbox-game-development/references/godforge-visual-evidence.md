# Godforge visual evidence — serial combat and consequence passes

## Event-to-cue map

- `combat` event: `attackerHumanId` / `targetHumanId` support anticipation, impact, recoil, and entity hit reaction. Keep the cue cosmetic; do not move authoritative positions or alter hitboxes.
- `human-died` event: `deadHumanId` supports a short ash/ground marker. The renderer must not remove the human; lifecycle remains simulation-owned.
- `activePowerEffects[powerId=fire]`: `affectedTileIndices`, `startsAtTick`, and `expiresAtTick` support deterministic fire visuals. Resolve tile positions from `world.map.tiles`; derive flicker from `world.tick` plus tile coordinates.

## Timing and quality

Use fixed tick windows, not wall-clock time, for event-driven visuals. Keep `reduceMotion` as a static high-contrast fallback. For Low quality, reduce geometry to a marker/dot rather than hiding the consequence entirely.

## Verification ceiling

A browser screenshot of the initial world proves baseline readability and console cleanliness, not that a transient fire/death/combat event was visibly active. Separate evidence into: source/typecheck/build coverage, reachable interaction coverage, and active-event screenshot coverage. If active-event evidence is unavailable, report it explicitly and do not overclaim.

## Closure pattern

For each serial pass: inspect source → patch one visual class → targeted typecheck/build → browser screenshot + console → full gates → commit/push → GitHub Actions status → stop. Do not bundle growth, resources, weather, and combat polish into one pass.
