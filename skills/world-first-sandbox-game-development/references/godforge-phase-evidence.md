# Godforge phase evidence notes

## Verified renderer pitfall

Godforge uses cached Pixi layers for performance. `renderPixi()` initially adds terrain, scenery, village details, entities, and impact in that order. Static redraws may call `app.stage.addChild(terrain/scenery/villageDetails)` again; Pixi moves those layers to the top, hiding dynamic humans and creatures even though the simulation and HUD counts are correct. After static redraw, restore the complete order: `app.stage.addChild(layers.villageDetails, layers.settlementLabels, layers.entities, layers.impact)`. Static content can be hidden too, not just entities, if the cached layer order is incomplete.

## Verified camera pitfall

The world is larger than the initial assumed viewport. For seed 42, the first capital is at `{x:24,y:24}` while the map is 128 tiles wide. Starting at world center with low zoom makes the settlement disappear into a debug-looking tile map. Start Sandbox camera on the capital with a meaningful zoom, then verify browser visuals.

## Verified power interaction pattern

Keep one `pointerToTile()` conversion and one `castPowerAt()` handler shared by click and drag. Maintain separate `brushTarget` (hover preview) and committed `target`. A drag must set a suppress-click flag so pointerup cast is not duplicated by the subsequent click event. Escape clears selected power, brush preview, and target.

## Verified living-world seed

The simulation can begin with zero creatures. Sandbox can seed a small neutral herd by calling the real `usePower()` spawn-animal command at the capital during Sandbox initialization, preserving Ranked/createWorld behavior while giving the player an immediately observable living world. HUD creature counts must read from the same `world.creatures` rendered by Pixi.

## Acceptance evidence

Use browser route `/`, Home → Enter Sandbox, inspect Canvas visually, check HUD counts, and inspect browser console. Then run `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, and `git diff --check` before phase closure.
