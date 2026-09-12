# Godforge Client Power-Roster Failure Prevention

Use this checklist whenever a client bottom dock or loadout exposes more powers.

## Rules

- Treat the client roster as a projection of `gameConfig.powers`, not a second catalog.
- Filter by the controlled god's compatibility and keep the equipped list within `minimumPowers`, `maximumPowers`, budget, and ultimate limits.
- Keep the equipped `powerNames` list and category tabs consistent. A visible button that is not equipped will be rejected by `usePower()`.
- Add localization for each visible power ID in EN/TH; raw IDs such as `tornado` are not acceptable UI labels.
- After roster edits run web-game typecheck, tests, build, then a fresh browser load. Vite build success does not catch mount-time `createCompetitiveMatch()` `RangeError` that leaves React blank.
- If the browser is blank, inspect console exceptions and `validateLoadout()` before weakening simulation validation. Fix the client fixture/roster, not the authoritative constraint.

## Verified failure pattern

A seven-power client fixture exceeded the six-slot loadout maximum and caused a mount-time blank page. Reducing to a valid roster and removing an un-equipped tab entry restored the page. A subsequent browser check showed the new Chaos tab and Tornado button, and the browser console was clean.
