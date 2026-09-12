---
name: game-ui-polish
description: Use when polishing game UIs.
license: MIT
metadata:
  author: Hermes Agent
  version: "1.0.0"
---

# Game UI Polish

Use this skill when a playable browser game needs a visual/usability pass rather than a new game system. Optimize for high-impact, rendered improvements: hierarchy, HUD readability, action feedback, onboarding, responsive layout, and accessibility. Preserve the existing visual identity and avoid speculative feature expansion.

## Workflow

1. **Audit the rendered surface first.** Run the app, inspect Home, Lobby, Play, and Settings in a real browser, and capture screenshots. Do not score a UI from source alone.
2. **Separate polish from behavior.** Keep the existing simulation/network contract intact. Prefer presentation state, existing config metadata, CSS, and existing localization keys over new domain systems.
3. **Prioritize the largest visible gaps.** In a game prototype, the usual order is: canvas/sidebar balance; game status strip; selected-power feedback; energy/cooldown metadata; target/cast confirmation; onboarding CTA hierarchy; responsive/mobile layout; focus-visible states.
4. **Use existing data.** Read power cost/cooldown from the config catalog, simulation tick from the current world, and the selected locale dictionary for every visible string. Never duplicate authoritative values in UI literals.

- Audit non-power surfaces separately from power controls. Search the screen union, render branches, navigation arrays, and CSS `display:none`/`visibility:hidden` rules. A screen that has JSX and state but is absent from navigation is hidden-by-routing, not missing implementation: expose the existing route first (for example, add an existing Shop screen to top nav and the Play dock) before building anything new. Verify Home → Lobby → Play → Loadout → Profile → Shop → Settings in a real browser and distinguish `implemented-but-inaccessible` from `not-implemented` (replay, spectator, minimap, history/event log, and real commerce should remain deferred if no runtime producer/consumer exists).
- Make feedback explicit. A click on a target or power should produce a visible state change (active button, target marker, status text, or short toast) and an audio route if the app already has an audio bus.
- For production audio/music, replace oscillator-based music beeps with bundled, loopable audio assets and one persistent `HTMLAudioElement`. Select tracks from existing presentation state (for example `home`, `lobby`, `play`, and result states), keep music and SFX volume controls separate, catch autoplay-policy rejections, and never couple audio state to authoritative simulation mutation. Add the relevant asset-module declaration (such as `*.wav`) and verify the production bundle contains the assets.
6. **Balance composed layouts.** When adding a right-side status panel, reserve its width in the left content column; constrain onboarding grids so cards cannot overlap or disappear beneath absolute panels. Add mobile overrides that return positioned panels to normal flow.
7. **Check accessibility as part of polish.** Add meaningful labels to canvas/custom interactive surfaces, preserve keyboard focus visibility, use live regions for transient status, and keep reduced-motion behavior intact.
8. **Verify in this order:** Prettier, focused UI package build, web-game typecheck/build, browser screenshot/smoke, then full format/lint/typecheck/test/build/diff-check. Report any HMR-only warning separately from production build errors.
9. **Score honestly.** A screenshot-based score is an estimate, not acceptance proof. Do not claim 9.5 merely because the build is green. Call out remaining prototype limitations such as missing cast animation, live cooldown state, art/audio assets, or interactive tutorial flow.
10. **Commit only when requested or explicitly authorized.** Keep UI polish changes uncommitted if the user asked for implementation/review only; when authorized, commit, push, and verify remote SHA plus clean working tree.

## Implementation patterns

- Prefer a single token/translation lookup in render logic over repeated locale conditionals.
- Prefer CSS composition for hierarchy and spacing; do not add a new dependency for a card, status strip, toast, or responsive grid.
- Use short-lived toast/status feedback with `role="status"` and a bounded timeout.
- Use a compact status strip above the game canvas for simulation readiness, match time, and target coordinates.
- Keep power controls compact but expose selected-power metadata such as energy cost and cooldown.
- For hero/home surfaces, a small system-status panel can use empty space productively, but it must reserve layout width and never overlap onboarding cards.
- For Pixi/canvas worlds, add scenery in one deterministic `Graphics` layer after terrain and before entities; derive decoration stamps from tile coordinates plus seed, and use silhouettes/shadows for characters and settlements so visual polish does not alter simulation state.
- Set a readable initial camera zoom when tile-level detail is otherwise invisible, but preserve pan/scroll zoom controls and verify the cropped composition in-browser.
- For a lightweight target/cast pulse, keep a single `{ position, startedAt, color }` presentation state, render it from the existing Pixi ticker, and expire it by elapsed time; do not add a new animation system or mutate simulation state.
- PixiJS v8 alpha fills should use object syntax (`fill({ color, alpha })`), not the deprecated two-argument form. After canvas interaction, clear the browser console and re-run the click once to verify no deprecation warnings.
- When the user asks to push a visual score higher, implement the smallest real feedback loop first (target pulse/impact confirmation) and state the ceiling honestly if authoritative cast events or live cooldown state are not wired yet.
- For character polish, reuse simulation fields already present on each human (`role`, `state`, `health`, `maxHealth`, faction) to render deterministic idle motion, role silhouettes, state markers, shadows, health bars, and target proximity highlights; do not invent a second character model or mutate simulation state.
- Treat “10/10” as a requested target, not evidence: report a screenshot-based estimate and name the remaining art/animation ceiling when procedural silhouettes are still being used.
- When character visuals are still procedural, add a compact HUD overview from authoritative simulation fields: living population, role split, average health, localized labels, and readable role dots. Keep it as a presentation-only card; do not invent character data or mutate simulation state.
- Character HUD and canvas detail should be reviewed together: the canvas communicates individual silhouette/state, while the HUD communicates aggregate counts/health. Verify the card in the real Play layout for overflow and mobile stacking.
- For an organic sandbox pass, render deterministic layers in order: terrain, seeded scenery, village roads/paths and clustered houses, settlement markers, dense entities, then target/pulse overlays. Derive decoration from `world.seed` plus coordinates; never use `Math.random()` or mutate simulation state. See `references/organic-pixel-world.md` for the research notes and validated layer recipe.
- When the target is a living pixel world, increase readability and inhabited density before adding more HUD cards: village clusters, faction roof accents, visible paths, entity silhouettes and quality-controlled density are higher-value than decorative particles.
- Make the canvas the visual focus: use a full-size canvas with a translucent scrollable desktop HUD overlay, then return the HUD to normal flow for mobile. Always inspect the real screenshot because build success does not prove composition.
- For this user, keep implementation-first updates terse (code/result first, no feature tour); provide a fuller report only when explicitly requested.

## God-game bottom-dock pattern

For world-first sandbox games where the canvas is the primary surface, a bottom dock is a strong alternative to a top navigation bar. Use a three-part hierarchy: category tabs, a horizontally scrollable power tray for the active category, and compact navigation/actions at the edge. Borrow the interaction pattern from genre references, not their assets, names, branding, distinctive palette, or exact layout.

Implementation rules:

- Keep the dock presentation-only: selecting a tab changes UI state and selected power, while casting continues through the existing authoritative handler.
- Derive power labels from existing localization/config data; do not create a second authoritative power catalog.
- Use a clear selected state with a high-contrast accent, icon + label, keyboard focus, and `aria-label` for icon-only actions.
- Hide duplicate desktop power controls after the dock is introduced; duplicate controls create state ambiguity and waste canvas space.
- Lock the desktop Play surface to the viewport (`position: fixed; inset: 0; overflow: hidden`) when the dock must remain anchored to the bottom. Restore normal document flow on mobile so touch users can scroll.
- Reserve dock space or constrain the canvas before visual review. Verify `document.documentElement.scrollHeight === window.innerHeight` on desktop; a build passing is not evidence that the dock is actually in the viewport.
- Run a real browser screenshot after HMR/full reload and inspect canvas, HUD, dock, and top hint/status text together. Remove or relocate duplicate hint text if it competes with the canvas/topbar.

The implementation evidence and viewport containment fix are recorded in `references/bottom-dock-browser-evidence.md`.

## Gameplay Closure Pattern

For a god-simulator UI, visual polish is not complete when the bottom dock renders. Close one playable vertical slice before adding more powers:

1. **Observe:** provide an explicit inspect mode separate from cast mode; click the world and read authoritative unit/settlement data.
2. **Choose:** category tabs and power buttons update presentation selection without mutating simulation state.
3. **Preview:** show the current target/brush position; derive valid/invalid styling from the authoritative map tile.
4. **Cast:** route through the existing authoritative `usePower`/intent handler; do not duplicate power rules in React.
5. **Consequence:** update the world, show target/impact feedback, and provide bounded status/toast feedback for success or invalid/empty targets.
6. **Re-observe:** keep inspection usable after world changes so the player can verify consequences.

Use a small pure helper for nearest-object inspection. Test living-human precedence, settlement fallback, dead-entity exclusion, and no-result behavior first. If a fixture contains both a civilization aggregate population and one living human, assert the aggregate field deliberately; do not silently substitute a filtered render count. Verify the complete flow in a real browser, not only with unit tests.

## Pitfalls

- A root dev command can appear to work while its path selector matches no workspace packages. Prefer explicit package selectors when the workspace resolver is unreliable; verify all intended app processes start.
- Native `<select>` interaction may not be fully controllable through every browser automation layer; verify its rendered options and use a real browser/manual pass for locale changes.
- Avoid replacing existing CTA blocks while inserting status panels; patch the smallest JSX region and re-run typecheck immediately.
- A UI build can pass while a screenshot reveals overlap. Always perform a visual pass after layout changes; specifically check brand/status, status strip/HUD, and floating controls/canvas bounds at the real viewport size.
- For a world-first HUD pass, hide non-gameplay metadata from the play top bar (match/quality/debug labels), reserve the sidebar width in status strips, and move pause/speed into one compact floating control group. Keep only actionable status such as targeting; duplicate match timers/readiness telemetry add prototype/debug weight. Re-capture after CSS changes because a successful build cannot prove layer separation.
- When a secondary HUD section is useful but visually too tall, use native `<details>/<summary>` for Character/Capital or other inspection-only data before inventing modal state or a custom accordion. Keep primary energy, population, and power selection always visible.
- Make a small power palette with equal-sized buttons and a clear active state; a text-button row with uneven widths reads as prototype even when behavior is correct. Use existing power labels/config, not new icon infrastructure, unless icons are already present.
- When validating Thai completeness, parse only the `th-TH` dictionary block (not the preceding `en-US` block) and distinguish technical object keys such as `spawn-animal` from rendered values. Search rendered JSX for literals separately.
- Do not use procedural oscillators or a status label as a substitute for real game feedback when claiming production readiness.
- Canvas detail can be technically present but visually unreadable at fit-to-world zoom; inspect the screenshot at the actual default camera scale before deciding the art pass is effective.
- Hot-module replacement can briefly leave the browser blank after a large JSX/Pixi edit; check console, then perform a full page reload before treating it as a runtime failure.
- Run Prettier after the final patch and before the full gate; a formatting-only failure should be fixed, not bypassed.

## Reference

See `references/godforge-ui-polish.md` for the validated patterns and evidence from the Godforge pass.
