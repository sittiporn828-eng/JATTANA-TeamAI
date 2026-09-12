# Authored Pixi asset integration

Use this when moving from procedural primitives to reviewed PNG assets.

## Verified workflow

1. Generate or obtain one isolated asset per role; do not import a composite target sheet into runtime.
2. Preprocess white-background references into real `RGBA` PNGs with bounded crop and transparent background. Verify mode, alpha extrema, dimensions, and file size before coding.
3. Keep assets in the web app source tree and add a single `declare module "*.png"` typing file if the repository does not already provide one.
4. For Pixi v8, load Vite-imported asset URLs with `Assets.load(url)` and assign the resolved `Texture` before constructing sprites. `Texture.from(url)` alone can leave the asset uncached and produce a blank sprite or Pixi asset-cache warning.
5. Treat textures as nullable until loading completes; guard sprite creation rather than constructing a sprite from an empty/undefined texture.
6. Roll out authored assets by quality tier when the asset is more detailed or heavier than the existing renderer: keep Medium/Low procedural until High-preset browser evidence proves the sprite reads at gameplay scale and does not regress performance.
7. Put authored sprites in a dedicated dynamic container. Rebuild that container with the dynamic render pass, but preserve the intended stage order: terrain → scenery → village details → labels → procedural entities → authored entities → impact/effects.
7. Remove procedural geometry only after the authored sprite has a verified shadow/grounding cue. Keep simulation-derived position, visibility stride, target state, and hit behavior unchanged.
8. Browser verification must show the actual sprites, transparent edges (no white box), readable scale, correct friendly/hostile/capital distinction, and zero console errors. Build success is insufficient.

## Pitfalls

- A generated image URL can expire or return 404; download and verify the local artifact before integration.
- An image that visually appears isolated may still be RGB with a white background; inspect `RGBA` alpha rather than trusting the preview.
- Re-adding only some Pixi layers after a cached-layer redraw can hide dynamic entities; inspect the actual screenshot after every stage-order change.
- Do not commit a runtime asset pass while browser verification still reports exceptions, even if typecheck and build pass.
