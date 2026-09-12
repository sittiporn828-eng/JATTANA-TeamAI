# Godforge Phase 4 Handoff

Verified from `docs/Godforge_Master_PRD_TDD_Phase_Based_v2.2.md` after Phase 3 completion.

## Next phase

**Phase 4 — Client Rendering & UX Core**.

Goal: make the deterministic simulation playable and understandable in the browser.

## Scope

- React for Main Menu, Lobby, Profile, Loadout, and other non-world UI.
- PixiJS for terrain, NPCs, buildings, effects, God Powers, camera, and overlays.
- Keep high-volume NPC rendering out of React DOM.
- Separate the 60 FPS client render loop from simulation state/update logic.
- Use interpolation between simulation states.
- Provide Low/Medium/High presets controlling particles, animation, render distance, effect quality, entity detail, and resolution scale without changing gameplay.
- HUD: match time, divine energy, power cooldowns, civilization score, population, territory, capital status, team, and God loadout.

## Acceptance gate

- World renders.
- Camera supports zoom and pan.
- Large NPC populations remain camera-controllable.
- Power targeting is understandable.
- HUD stays synchronized with simulation state.
- Low preset reduces graphics load without changing gameplay.

## Handoff rule

Before starting Phase 4, preserve the Phase 3 simulation contract and run the producer build plus full test suite. Treat rendering as a client projection of authoritative simulation state; do not duplicate gameplay rules in React/PixiJS.

## Verified closure recipe

1. Add `@godforge/simulation` as a workspace dependency of `apps/web-game`; build the producer before client typecheck/tests so workspace `dist` is fresh.
2. Use PixiJS `Application` on the client canvas, `Graphics` for terrain/entities, and a Pixi ticker for 60 FPS rendering. Keep React for shell/HUD/menu surfaces only.
3. Keep `previousWorld` and `currentWorld` refs; interpolate entity positions with `previous + (current - previous) * alpha` between 12 TPS simulation updates.
4. Smoke-test the running browser: Home, Lobby, Play canvas, Loadout, Profile, Shop, Settings, power selection/targeting, quality preset, language, reduce-motion, and colorblind toggles. Check browser console and shut down the dev server.
5. Run `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build`. A full Phase 4 claim requires all PRD UI surfaces plus this gate; a world-render-only result is a baseline.

The implementation that prompted this handoff exposed two recurring review traps: a green simulation suite does not prove UI acceptance, and a fast vertical slice must not be reported as a complete phase when menu/settings/localization surfaces are still absent.
