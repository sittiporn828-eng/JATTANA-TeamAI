---
name: world-first-sandbox-game-development
description: Use for serial evidence-driven sandbox game development.
---

# World-first sandbox game development

Use this for Godforge-style work where the target is a living, inspectable sandbox world inspired by god-game principles without copying another game's assets, code, branding, or layout.

## Operating contract

1. Read the PRD, current phase, relevant renderer/simulation flow, and git state before editing.
2. Work strictly serially: select one phase, define acceptance, implement only that phase, verify it, commit/push it, then stop.
3. Treat the world canvas and simulation as primary. HUD is supporting instrumentation, not the product surface.
4. Never claim a feature works from a build alone. Require browser runtime evidence and console inspection.
5. Preserve deterministic simulation, en-US/th-TH localization, accessibility, game-server lifecycle, and existing quality gates.
6. Do not copy WorldBox assets, sprites, code, branding, or layout; borrow only high-level gameplay principles such as observe, intervene, and watch consequences.

## Phase method

For each phase:

- State the single phase goal and explicit acceptance checks.
- Trace the existing path end-to-end before writing: simulation state → render layer → input → HUD feedback.
- Prefer reuse and the smallest diff; avoid speculative abstractions and placeholder systems.
- Implement the smallest real behavior, not a visual mock. Effects must mutate simulation state when the phase requires gameplay.
- Run targeted typecheck/build early.
- Run browser smoke/visual checks on the actual route and inspect console errors/warnings.
- Run the complete project gates before closure:
  `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `git diff --check`.
- Commit with a phase-scoped message, push only after verification, confirm local/remote HEAD and clean working tree.
- Stop and report the completed phase; do not begin the next phase without a new instruction.

## World-first acceptance

A phase is not complete if the screenshot still reads as a debug tile map or dashboard when the phase is about world presentation. Check:

- initial camera focuses a meaningful settlement/landmark, not empty map center;
- biome separation is readable without relying on HUD text;
- settlements, roads, units, and creatures are visible at the intended zoom;
- dynamic Pixi layers remain above static terrain/scenery layers;
- runtime counts and visual entities come from the same simulation state;
- direct power interaction has target preview, validation feedback, and visible consequence when in scope.

## Performance discipline

- Keep static terrain/scenery cached; update dynamic entities/effects separately.
- Use deterministic `seed + position` placement.
- Use quality-aware strides only as a measured fallback; do not silently hide most of the living world merely to hit FPS.
- Benchmark Medium and High in a real browser; record average FPS and frame time.
- When static Pixi layers are re-added, explicitly restore the complete intended order (`terrain → scenery → village details → labels → entities → effects`); otherwise entities, houses, labels, or territory rings can render in state but disappear behind terrain.
- After any stage-order or cached-layer change, use a real browser screenshot to verify both static and dynamic content—not only HUD counts. A nonzero simulation count is not visual evidence.
- Include semantic settlement/civilization state (stage, health, city count) in the static-layer cache key so labels, territory rings, and city landmarks redraw when the simulation evolves.

## Interaction discipline

- Share one coordinate conversion and one cast handler between click and drag paths.
- Keep hover/brush preview separate from committed target state.
- Drag-to-pan must not accidentally cast; drag-to-paint must intentionally cast at the end/brush path.
- Escape clears selected power, preview, and target.
- Error feedback must be visible and localized where practical.

## Serial visual-polish loop

When the user asks to remove remaining prototype feel, fix one observed defect per turn/commit rather than bundling a redesign:

1. Capture the current Play route in a real browser and name one concrete defect.
2. Prefer deletion or native CSS/HTML first: remove duplicate telemetry, use `<details>` for secondary HUD, and size overlays to content instead of viewport height.
3. For world readability, modify the existing renderer layers only; keep simulation state authoritative and deterministic. Small silhouettes, shadows, faction cues, and deterministic terrain stamps beat speculative asset systems.
4. If the map reads as noisy speckle, improve composition at the existing generator boundary before adding art: blend a coarse region hash with a fine tile hash (for example, weighted coarse 6×6 regions plus fine detail) so biomes cluster while retaining seeded variation. Keep terrain types, API shape, spawn-safe zones, and replay invariants unchanged.
5. Re-run browser visual smoke after each change. Inspect for overlap, clipped controls, missing static/dynamic layers, and console errors before full gates.
6. Only then run the full gates and commit/push; confirm clean working tree and local HEAD equals origin.

### Verified renderer/generator patterns

- When Medium/High world entities look like isolated dots, reuse the current draw layer and add minimal silhouettes, heads, faction colors, and deterministic shadows; do not introduce an asset pipeline just to improve prototype readability.
- Quality-aware entity strides are acceptable only as a measured fallback: Medium can render all creatures when the current world size allows it, while Low may retain a reduced stride. Re-check browser readability after changing the stride.
- A full gate run is `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && git diff --check`; Vitest does not accept Jest's `--runInBand` flag, so invoke the repository test script without that flag.
- For lightweight production polish, derive feedback from authoritative tick/state already available in the renderer (for example, a small settlement smoke offset from `world.tick` and settlement position). Keep it deterministic, bounded, and subordinate to landmark readability; do not add a particle system or asset pipeline for one feedback cue.
- After adding a dynamic cue, verify both normal and reduced-motion behavior in-browser; console cleanliness plus a screenshot must confirm the cue does not obscure settlements, capital markers, entities, or HUD.
- For procedural art polish, add one static-layer detail at a time (for example, shoreline edge markers derived from neighboring terrain). Keep it inside the existing cached scenery layer; do not add a per-frame particle system for static decoration.
- Measure performance after renderer polish with a 10-second `requestAnimationFrame` sample. Report average/min FPS and max frame time separately; a single 60 FPS run does not justify hiding a small frame spike or claiming all hardware targets pass.
- `references/ui-polish-evidence.md` records the compact HUD, duplicate-targeting removal, deterministic entity/terrain readability changes, and browser evidence pattern.
- `references/procedural-art-performance.md` records the validated shoreline pass and the before/after browser frame metrics.
- `references/authored-pixi-assets.md` records the verified workflow for transparent PNG preprocessing, Pixi asset loading, dynamic layer order, and browser-first acceptance.
- `references/authoritative-art-class-closure.md` records the verified serial class workflow for snapshot validation, exact fail-closed selectors, static Pixi caching, child destruction, browser/FPS evidence, review, and commit closure.
- `references/combat-feedback-evidence.md` records the validated event-to-renderer workflow for deterministic combat impact, entity recoil, and death feedback.
- `references/godforge-visual-evidence.md` records the serial event-to-cue map, fire consequence rendering, quality fallbacks, and the evidence ceiling for transient effects.

Avoid claiming “done” from source grep or build output alone. Browser evidence must show the intended world presentation, not merely nonzero HUD counts.

## Returning a local playable stack for manual testing

When the user is returning to playtest, prepare the existing stack rather than changing code: run the repository `pnpm dev` script, inspect its actual Vite/API/game-server output, and probe API and game-server health endpoints before reporting a URL. The web game defaults to Vite `5173`, but Vite may move to the next free port; report the port printed by Vite, not the default. Confirm the API and authoritative game server respond successfully, report any port collision or fallback explicitly, keep the dev process running for the user's manual session, and do not start unrelated services or modify source just to make the test convenient.

## Closure report

Report only what was actually verified:

- phase and commit;
- files/behavior changed;
- browser evidence and console result;
- full gate result;
- known simplifications or deferred work;
- explicit statement that the next phase was not started.

## References

- See `references/godforge-phase-evidence.md` for concrete renderer/input pitfalls and verified checks from prior implementation.
