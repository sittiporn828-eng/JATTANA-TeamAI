# UI polish evidence patterns

## Verified sequence

- Inspect the live Play route before editing; browser screenshots catch overlap and hierarchy defects that source inspection misses.
- Fix one visual defect at a time: duplicate status → HUD height → world entity readability → deterministic terrain detail → biome composition.
- Native platform primitives were sufficient: remove duplicate markup, use `<details>` for secondary Character/Capital sections, and remove `bottom` from the absolute HUD so it ends with content.
- Keep the canvas primary. Power controls can remain text buttons if they are normalized into a compact equal-width palette; do not add icons/assets without an authored-art need.
- In the existing Pixi renderer, improve readability with deterministic geometry: human head/body/shadow silhouettes, creature stride adjustments, and seeded terrain stamps. Preserve simulation data, `seed + position` placement, and quality-aware fallback for Low.
- If terrain is noisy speckle, improve the existing generator boundary instead of adding art: blend a coarse region hash (6×6 in this session) with a fine tile hash, weighted toward the region hash. This clusters biomes while retaining deterministic variation without changing terrain types or API shape.

## Acceptance evidence

- Browser snapshot showed one `Targeting` readout under God Loadout, working Pause/1x/2x/4x and power buttons.
- Visual smoke showed terrain, trees, rocky details, water, capital, settlement, creatures, and readable human silhouettes without HUD/canvas overlap.
- Browser console reported zero errors after the Play route loaded.
- Full closure gates used: `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `git diff --check`.
- Repository tests passed with API 99, game-server 4, and testing 37 in this session.

## Pitfalls

- Do not treat a successful build or HUD population counts as proof that entities are visually rendered.
- Do not remove quality-aware strides globally; Medium/High can show more world detail while Low remains a performance fallback.
- Do not bundle terrain-generation or authored-art work into a small HUD polish phase; defer authored art until visual direction/assets are explicit.
- Vitest does not accept Jest's `--runInBand`; use the repository's test command without that flag.
