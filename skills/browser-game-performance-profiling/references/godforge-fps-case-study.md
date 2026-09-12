# Godforge FPS Case Study (2026-08-11)

## Reproduction

- Repository: `C:\Users\Acer\Desktop\JATANA GROUP\Godforge`
- Start: `pnpm --filter @godforge/web-game dev`
- Browser: local Vite Play screen, same viewport and seeded world
- Measurement: browser `requestAnimationFrame` promise for 10 seconds; discard first five samples
- Report: frames, duration, average/min/max FPS, average/max frame time

## Findings

The pre-optimization Play renderer rebuilt the whole Pixi scene on every ticker callback with `removeChildren().destroy()` and new `Graphics` objects. The measured Medium result was **6.42 FPS**, so build/test/smoke success was not evidence of playability.

A first persistent-layer experiment that still replayed too much work regressed to **4.66 FPS** and was reverted. Keep failed experiments out of the shipped change.

## Validated fix pattern

1. Keep persistent Pixi layers for terrain, scenery, settlements, entities, and impact effects.
2. Cache static drawing behind a dirty key containing viewport, camera, preset, and world seed.
3. Clear/redraw only dynamic entity and effect layers per frame.
4. Use lightweight faction silhouettes for Medium/Low; keep full detail for rendered High entities.
5. Benchmark each preset independently; for High, increment entity stride only as needed and retain full detail per rendered entity.

## Verified results

- Medium after static caching + lightweight Medium entities: **60.002 FPS**, `16.666 ms` average frame time over `10.001 s`.
- High initial after static caching: **46.31 FPS**, max frame time `50 ms`.
- High with stride 2: **59.30 FPS**; failed an explicit 60 FPS gate.
- High with stride 3: **59.90 FPS**; still failed without rounding.
- High with stride 4: **60.002 FPS**, `16.666 ms` average frame time, `597` active frames over `10.0017 s`, min about `59.52 FPS`, max `60.61 FPS`, max frame time `16.8 ms`.

Functional Play smoke and browser console were checked separately; the console had zero messages/errors in the final High run. Full format, lint, typecheck, tests, build, and diff checks passed before shipping.

## Reporting rule

When the user says “at least 60 FPS,” do not call `59.x` a pass. State the preset, entity sampling/stride, browser context, and the exact measured values. A single-machine result is evidence for that context, not a universal hardware guarantee.
