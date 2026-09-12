# Combat feedback evidence

Validated Godforge combat-presentation workflow:

- Keep authoritative combat in `packages/simulation`; renderer-only polish belongs in `apps/web-game`.
- Trace event schema before visual work. Civilization-level `combat` events can drive battle-front cues; entity recoil requires authoritative `attackerHumanId`/`targetHumanId` fields.
- Add entity IDs at event creation, not by guessing from current positions. For death, emit `human-died` only on the transition `health > 0` to `health === 0`, with `deadHumanId`; do not let the renderer remove entities.
- Derive all cue timing from `world.tick - event.tick`. Use bounded windows (anticipation/impact/reaction/death) and keep `reduceMotion` static rather than removing semantic feedback.
- Reuse existing Pixi `Graphics`/dynamic entity layers. Procedural wedge, ring, shards, recoil, ash ellipse, smoke ring, and cross markers are enough before an animation atlas is justified.
- Presentation transforms must not change simulation position, collision, health, damage, replay ordering, or server authority.
- Validate in this order: targeted package build/typecheck/tests (build simulation before consumers when declarations are stale), browser route + console + screenshot, then full gates and CI.
- Full closure evidence used successfully: `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `git diff --check`; commit/push; verify GitHub Actions success and clean local/remote HEAD.

Known ceiling: current event model supports deterministic feedback but not per-frame authored attack/death animation. Add an atlas only when multiple states/frames and a performance target justify it.
