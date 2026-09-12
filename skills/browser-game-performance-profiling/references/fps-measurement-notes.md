# FPS Measurement Notes

## Validated browser probe

Use the measurement snippet in the parent `SKILL.md` after the Play canvas is visible. A 10-second sample is long enough to expose stalls while remaining practical during a smoke run.

## Interpretation

`requestAnimationFrame` measures the browser's delivered frame cadence, not a game's simulation tick. Report both only when both are measured. A build, screenshot, or successful click does not establish FPS.

## Required report shape

- Preset and viewport
- Sample duration and frame count
- Average / min / max FPS
- Average / max frame time
- Functional smoke result
- Console result
- Whether any optimization was shipped or reverted

## Validated Godforge case

The Pixi renderer initially rebuilt the full scene from the ticker callback. A direct Medium-quality browser sample measured **6.42 FPS average**, **5 FPS minimum**, **7.51 FPS maximum**, **155.7 ms average frame time**, and **200 ms maximum frame time** over 10.09 seconds. A naive persistent-layer experiment measured **4.66 FPS** and was reverted; persistent layers alone are not sufficient.

The working optimization was validated in the same Play flow and Medium preset:

- cache terrain/scenery/settlement Graphics layers by viewport, camera, preset, and world seed;
- clear/redraw only dynamic entities/effects each frame;
- use lightweight faction silhouettes for Medium/Low while retaining full character detail on High.

Final browser probe: **60.00 FPS average**, **59.52 FPS minimum**, **60.61 FPS maximum**, **16.666 ms average frame time**, **16.8 ms maximum frame time**, **596 frames over 10.001 seconds**. Browser visual smoke passed and console reported zero errors/warnings. Full repository gates also passed before the optimization was committed.

These values are session evidence for this machine, browser, viewport, world, and preset—not universal hardware guarantees. Re-run the probe after renderer or quality-preset changes. High preset was not established by this sample and must be benchmarked separately before claiming a High-preset target.
