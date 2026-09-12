# Procedural art + performance evidence

## Validated pass

A minimal shoreline pass improved water readability without adding an asset pipeline:

- Build on the existing cached `scenery` layer.
- Build a tile-position lookup from the authoritative map.
- For each water tile, inspect four neighbors and draw a small deterministic edge marker where terrain changes.
- Keep the pass inside `staticDirty`; do not redraw it every ticker frame.
- Preserve seed-based rendering and existing terrain/entity layers.

## Verification

Browser Play smoke showed:

- water edges visually separated from grass/forest/rocky terrain;
- entities, capital, settlement, HUD and power palette still readable;
- no overlap;
- browser console had zero errors.

A 10-second Medium-preset `requestAnimationFrame` sample after the pass reported approximately:

```text
avg FPS: 59.70
min FPS: 30.03
max frame time: 33.30ms
frames: 594
seconds: 10.007
```

Baseline before the pass was approximately 60.00 FPS, 16.80ms max frame time. Treat this as near-60 with a visible single-frame spike, not proof of universal 60 FPS across hardware. Do not hide the spike by changing thresholds or adding a build warning suppression.

## Rule

Static visual detail belongs in the existing cached layer first. If a future art pass causes sustained frame loss, measure High and Medium separately, identify the hot layer, then cache or stride that layer before adding more effects.
