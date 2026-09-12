# Godforge power interaction case study

## Failure pattern

A Play screen can look functional while powers do nothing: the button only calls `setSelectedPower`, and canvas click only calls `setTarget`, creates a pulse, and shows a toast. This is target-preview feedback, not gameplay.

## Tight repro

1. Start the web game with `pnpm --filter @godforge/web-game dev`.
2. Open `http://localhost:5173`, enter Play.
3. Select a power, click a traversable world tile.
4. Verify more than the target ring/toast:
   - energy decreases by the configured cost;
   - cooldown is recorded;
   - world state changes (terrain, creatures, humans, or active effects);
   - invalid targets show an explicit failure.
5. Check the browser console for errors.

## Validated fix pattern

Use the domain simulation command from the UI event. For Godforge this is `usePower(activeMatch, { playerId, powerId, target })`. On success, replace the client world with `result.match.world`, retain the returned match/player state for energy and cooldown, and render the impact feedback. On failure, show the returned error rather than pretending the target was cast.

Keep the local UI loadout aligned with `createCompetitiveMatch` validation. If only a nature setup is used, expose only powers valid for that setup unless the client creates the corresponding player/god/loadout configuration.

## Evidence from the fixed flow

- Forest cast changed energy from `40` to `34` (cost `6`).
- The target coordinates were shown after a successful cast.
- Full format, lint, typecheck, test, build, and diff checks passed.

## Pitfalls

- A toast saying “target locked” is not proof of a cast.
- Showing six powers while the local competitive setup equips only four creates misleading controls; filter the UI or configure a valid loadout.
- A client-only local match is useful for a playable preview, but it is not server-authoritative multiplayer. Keep the distinction explicit until the WebSocket power intent path is wired.
