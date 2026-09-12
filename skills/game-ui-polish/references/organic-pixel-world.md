# Organic Pixel-World Reference

## Research-derived patterns

Public WorldBox materials describe a pixel-art sandbox built around creatures, civilizations, houses, roads, wars, powers, unit inspection, health/status feedback, minimap indicators, zoom controls, selected-power feedback, animated effects, and varied world templates. Use these as category-level design patterns only; do not copy sprites, icons, branding, toolbar layout, colors, code, or names.

Sources:
- https://play.google.com/store/apps/details?id=com.mkarpenko.worldbox&hl=en_US
- https://www.superworldbox.com/changelog
- https://www.superworldbox.com/

## Validated Godforge implementation pattern

For a deterministic Pixi world, render presentation-only layers in this order:

1. terrain tiles
2. seeded scenery (trees, mountains, water accents)
3. deterministic village details (roads, paths, house rectangles, roof triangles)
4. settlement/capital markers
5. humans/creatures with role silhouettes, shadows, state markers and health bars
6. target/pulse overlays

Derive village layout from `world.seed`, settlement coordinates and loop index. Never use `Math.random()` or mutate simulation state for decoration. Use quality presets to control entity density while keeping the same world state.

## Layout rule

Make the world canvas the visual focus. On desktop, use a full-size canvas with a translucent, scrollable HUD overlay; on mobile, return the HUD to normal document flow and stack cards. Check the real Play screenshot after any overlay change because a successful build does not prove composition.

## Acceptance

- houses cluster around settlements and use faction roof accents
- roads/paths are visible but subordinate to terrain
- humans/creatures are dense enough to make the world feel inhabited
- canvas remains readable under the HUD
- no console errors or Pixi deprecation warnings
- deterministic simulation/replay tests remain unchanged
