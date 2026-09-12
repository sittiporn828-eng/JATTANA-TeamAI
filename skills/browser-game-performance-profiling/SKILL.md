---
name: browser-game-performance-profiling
description: "Measure and fix real browser-game frame performance."
version: 1.0.0
author: นัท (sittiporn828), Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [browser, game-performance, fps, pixi, profiling]
    related_skills: [web-app-smoke-testing, systematic-debugging]
---

# Browser Game Performance Profiling Skill

Use when a browser game must be proven playable, when the user asks for real FPS numbers, or when visual smoke testing passes but the runtime may still be slow. This skill separates functional correctness from frame performance and requires a before/after measurement for every rendering change.

## When to use

- User asks "ตัวเลขจริง", FPS, frame time, smoothness, or whether the game is playable
- Canvas/WebGL/Pixi rendering has many entities, particles, tiles, or layers
- Browser smoke passes but input/render latency is suspected
- A visual polish pass changes the render loop

Do not use a screenshot alone as performance evidence.

## Procedure

1. Start the actual web app with the repository's workspace command and use the URL printed by Vite; do not assume a port.
2. Navigate to the real Play screen and select the intended quality preset.
3. Measure `requestAnimationFrame` for at least 10 seconds in the browser console. Report frame count, duration, average FPS, min FPS, max FPS, average frame time, and max frame time.
4. Repeat after the smallest candidate change. Keep the same browser viewport, screen, quality preset, and world seed when possible.
5. If FPS is low, trace the render loop end-to-end before editing: identify full-scene rebuilds, `destroy()`/allocation churn, synchronous simulation work, oversized draw calls, and repeated layout/style work.
6. Change one hot path at a time and rerun the same measurement. A regression is a failed iteration, not a reason to average it away.
7. Keep visual and functional smoke separate: verify canvas/HUD/power interaction and console errors, then report performance numbers independently.
8. Stop and report a blocker honestly when optimization is not verified; do not claim "playable" from a successful build or screenshot.

## Measurement snippet

Run this in `browser_console(expression=...)` after Play is visible:

```js
new Promise(resolve => {
  const samples = [];
  let last = performance.now();
  const start = last;
  function tick(now) {
    samples.push(now - last);
    last = now;
    if (now - start < 10000) return requestAnimationFrame(tick);
    const active = samples.slice(5);
    const avg = active.reduce((sum, value) => sum + value, 0) / active.length;
    resolve({
      frames: active.length,
      seconds: (last - start) / 1000,
      avgFps: 1000 / avg,
      minFps: 1000 / Math.max(...active),
      maxFps: 1000 / Math.min(...active),
      avgFrameMs: avg,
      maxFrameMs: Math.max(...active),
    });
  }
  requestAnimationFrame(tick);
})
```

## Acceptance thresholds

Use product requirements when they exist. Otherwise report these bands rather than inventing a pass:

- If the user explicitly requires 60 FPS, treat `avgFps >= 60` as the pass gate; do not round 59.x up. Also report min FPS and max frame time so a barely-passing average is visible.
- 55–59.99 FPS average: near-target but failed for an explicit 60 FPS requirement; inspect spikes and add headroom.
- 30–54 FPS average: usable but visibly below 60Hz; inspect spikes.
- 15–29 FPS average: degraded; fix before calling it polished.
- below 15 FPS average: performance failure for an interactive game.

A 60 FPS pass on one machine/preset does not prove every preset or hardware target. Benchmark each requested preset separately and state the tested browser, viewport, and hardware context.

Always include min FPS and max frame time; average alone can hide stalls.

## Rendering pitfalls

- A successful browser smoke test proves rendering and interaction, not frame rate.
- Rebuilding a Pixi scene every ticker callback (`removeChildren`, `destroy`, new `Graphics`) is a high-risk allocation hot path; inspect it before adding more visual detail.
- Persistent layers are not automatically faster: `Graphics.clear()` plus replaying thousands of draw commands can still regress. Measure the exact implementation and roll back if the number worsens.
- Cache static terrain/scenery/settlement layers by viewport, camera, preset, and world seed; clear/redraw only dynamic entity/effect layers per frame. A practical cache key is `[width, height, camera.x, camera.y, camera.zoom, preset, world.seed]`.
- Persistent layers alone are insufficient if every layer is still cleared or every static draw command is replayed. Guard static drawing behind a dirty-key check, then clear only dynamic entity/effect layers.
- For Pixi `Graphics` entity storms, use quality-aware rendering: keep full silhouettes/health/state details for High, but use lightweight faction silhouettes for Medium/Low. If High still misses an explicit 60 FPS gate, increase its entity stride incrementally (for example 1 → 2 → 3 → 4), keeping full detail for each rendered entity and measuring after every change.
- This is an explicit visual-density tradeoff: document the stride per preset and verify that terrain, settlements, roads, HUD, and interaction remain visible. Never claim that all entities are rendered when quality-aware sampling is active.
- Do not hide a slow renderer by throttling updates without stating the effective visual update rate.
- Do not change simulation tick, determinism, or quality semantics merely to make a benchmark look better.
- Browser console errors and performance regressions are separate failure channels; report both.

## Functional interaction gate

Performance work must not leave the game as a visual-only preview. For a god-game or canvas game, trace every control from UI event to state mutation:

1. Selecting a power should update the selected-power state.
2. Clicking the world should invoke the simulation/domain command (for Godforge, `usePower`), not only set a target or draw a pulse.
3. Verify the command result: world state changes, energy/cooldown changes, or an explicit localized error is visible.
4. Test at least two different powers and inspect the browser console.
5. If the UI intentionally exposes only a valid loadout subset, make the subset match the simulation setup; do not show powers that the local match cannot equip.

A successful click/toast or a target ring is not evidence that gameplay occurred. See `references/godforge-power-interaction-case-study.md` for the validated failure pattern and repro checklist.

## Godforge rendering-density pattern

For Godforge Pixi scenes, inspect authored entity stride before attempting a renderer rewrite. If a 10-second browser baseline shows a large creature-render cost, the smallest validated trade-off is to increase the creature stride per quality preset (for example High/Medium `1 → 2`) while preserving terrain, settlement, human, and interaction rendering. Re-measure the identical screen and preset immediately; in the validated pass this restored Medium from `34.99 FPS / 50 ms max frame` to `60.00 FPS / 16.8 ms max frame`, and High also measured `60.00 FPS / 16.8 ms max frame`. Report that not every creature is rendered at full density. Keep the change presentation-only and do not alter simulation cadence or determinism.

## Verification

- [ ] same screen, preset, viewport, and seed before/after
- [ ] 10-second runtime measurement completed
- [ ] average/min/max FPS and frame-time numbers recorded
- [ ] functional Play flow tested separately
- [ ] power selection + world click reaches the domain/simulation command
- [ ] world/energy/cooldown mutation or explicit error verified
- [ ] at least two powers tested
- [ ] console checked
- [ ] no unverified performance claim
- [ ] failed optimization experiments reverted or clearly isolated

## Related reference

See `references/fps-measurement-notes.md` for the validated measurement interpretation and reporting format. See `references/godforge-fps-case-study.md` for the Pixi static-layer caching and quality-stride case study.
