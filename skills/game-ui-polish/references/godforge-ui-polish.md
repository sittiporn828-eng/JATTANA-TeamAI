# Validated Godforge UI Polish Notes

## Evidence

- Home screenshot before polish had a large unused right side; onboarding was a three-card row with only two CTAs.
- Play screenshot showed a readable canvas/sidebar split but no explicit simulation status, target confirmation, or selected-power metadata.
- Browser smoke reached Home → Lobby → Play → Settings; Play rendered the canvas and six powers; browser console had no JS errors.
- Full gates after the polish pass passed: format check, lint, typecheck, tests, build, and `git diff --check`.

## Proven patterns

- A compact home status card (`World simulation online`, seed, config status, targeting state) filled the hero's unused space without adding a new product system.
- Reserving right-panel width with `padding-right` and constraining the onboarding grid prevented overlap; a later screenshot caught and verified this exact issue.
- A play status strip above the canvas made simulation readiness, match time, and target coordinates visible at a glance.
- Selected power metadata sourced from `powerCatalog` exposed energy cost and cooldown; a bounded `role=status` toast confirmed target selection.
- `aria-label` on the canvas and explicit `:focus-visible` styles improved keyboard/accessibility polish without adding dependencies.
- A deterministic scenery layer in the Pixi renderer added tree crowns/trunks, mountain facets, water lines, settlement roofs, and character head/body/shadow silhouettes without changing simulation data. The decoration stamp used tile coordinates plus seed, keeping replays stable.
- The initial camera was raised from `1` to `1.35` after a real screenshot showed that fit-to-world rendering made the new scenery too small to read; the follow-up screenshot confirmed clearer world detail with no glitches.
- Character polish should reuse the existing `Human` fields: faction color, `role` for soldier/civilian silhouette, `state` for a compact status dot and deterministic idle bob, and `health/maxHealth` for a small health bar. Target proximity can add a local highlight without changing simulation state.
- For lightweight cast/target feedback, a single `{ position, startedAt, color }` presentation pulse rendered from the existing Pixi ticker was sufficient; object alpha syntax (`fill({ color, alpha })`) avoids PixiJS v8 deprecation warnings.

- The character overview HUD card was validated in the Play screenshot: it showed living population, Soldiers/Civilians role split, and average health without overflow. It complements (rather than duplicates) the canvas's individual character silhouette/state rendering.

## Verification nuance

- After large JSX/Pixi edits, HMR briefly produced a blank browser page while console errors remained empty; a full navigation restored the app. Treat this as an HMR verification step, not as proof of a production runtime failure.
- After canvas interaction, clear the browser console and click once more; distinguish actual JS errors from library deprecation warnings, then fix deprecated API usage before claiming a clean smoke test.
- The final pass required running Prettier again before the full gates; the first final `format:check` caught only unformatted `main.tsx`, then the corrected full gate passed.
- A visual score must remain an estimate: screenshot evidence can support a polish assessment, but green gates do not prove a 9.5/10 production-art score.

## Known ceiling

This pass is still a prototype-level visual layer: real server-authoritative cast animations, live cooldown progression, richer art/audio assets, and an interactive tutorial are separate product work. Do not assign a 9.5 production score solely from build health or static cards.
