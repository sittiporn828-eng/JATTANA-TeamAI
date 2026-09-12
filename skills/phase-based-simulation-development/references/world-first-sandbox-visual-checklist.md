# World-first sandbox visual checklist

Use this after each world-presentation phase and before claiming completion.

## Source inspection

- Read `createWorld`/world constants and print the actual map width/height.
- Print settlement/capital positions and civilization ownership.
- Confirm the renderer uses the same `WorldState`; do not invent a second map.
- Verify the initial camera points at a real landmark, not blindly at `WORLD_WIDTH / 2`.

## Browser review

1. Start the real dev server and open the browser.
2. Enter Sandbox/Practice, not only a static preview.
3. Capture the default view at the actual target viewport.
4. Check biome separation: water, forest, grassland, rocky terrain.
5. Check the default focal point: capital, houses, roads, and nearby units must be readable.
6. Zoom out and confirm the wider world remains navigable.
7. Pan away and back; the same landmark must remain deterministic.
8. Check that HUD panels do not dominate the world at the phase's intended scope.
9. Inspect browser console; zero errors and warnings is required.

## Completion rules

- Build/typecheck/FPS are necessary but do not prove visual completion.
- If the screenshot is still a tile mosaic and settlements are unreadable, keep the phase open.
- Do not claim WorldBox parity from a palette change, a target ring, or procedural decorations alone.
- Report procedural art, missing authored sprites, missing living-world behavior, and oversized HUD as explicit later-phase limits.

## Godforge coordinate pitfall

Godforge's generated world is larger than the viewport (for example, `128 x 192`), while initial capitals can be near `(24,24)` and `(104,24)`. A camera initialized at the map center can make the capital appear absent even though the renderer is drawing it. Always inspect generated coordinates before tuning zoom or blaming settlement rendering.
