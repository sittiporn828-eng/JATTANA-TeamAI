# Authoritative world-art class closure

Use this when replacing procedural placeholders with authored Pixi assets while preserving deterministic/server-authoritative behavior.

## Serial class workflow

1. Pick one class only (vegetation, buildings, units, vehicles, VFX).
2. Trace its authoritative domain fields and consumers before drawing. Remove cosmetic fake state when equivalent authoritative entities already exist.
3. For a new visual selector, write exact mapping tests first and observe RED. Assert every supported domain variant explicitly; reject unknown faction/kind IDs rather than routing them to a default appearance.
4. If a renderer begins consuming a newly required snapshot array, update the trust-boundary validator in the same class and add a stale-snapshot regression proving the missing field is rejected before rendering.
5. Author isolated SVG sources and reproducible transparent PNG exports. Verify dimensions, real alpha, transparent corner pixels, and source→PNG pixel equivalence. Reject baked checkerboards, white fields, and composition-sheet crops.
6. Keep semantic Low fallback; Medium/High may use authored sprites only after browser readability proves the class.

## Pixi cache and lifecycle rules

- Static world geometry (vegetation, roads, buildings) belongs in a persistent static container. Include every field affecting appearance/position/state plus asset revision, camera, preset, and viewport in the existing dirty key.
- Clear/destroy and rebuild static children only when `staticDirty`; never destroy/recreate static sprites every render tick just because current FPS still looks acceptable.
- Dynamic humans, creatures, interpolation, state markers, and VFX remain outside static caches.
- Establish layer order once. Repeated `stage.addChild(existingLayer)` reorders children and can silently place terrain over entities.
- On Pixi v8 teardown, `Application.destroy(true)` does not destroy stage children. Pass child destruction explicitly: `app.destroy(true, { children: true })` on both async-abort and unmount paths.
- Separate buildings behind dynamic entities when one shared authored container would make insertion order determine occlusion.

## Acceptance evidence

- Exact selector/trust-boundary focused tests are RED→GREEN.
- Low, Medium, High browser screenshots show no boxes, duplicates, missing layers, road overlap, or incorrect occlusion.
- Capture an isolated 10-second FPS sample per required tier; report average, p95, max frame time, and transient spikes separately.
- Exercise screen transitions/unmount and require a clean console.
- Run full format/lint/typecheck/tests/build/diff-check.
- Run fresh fail-closed review after fixes; commit/push only when it passes. Close and publish one class before starting the next.
