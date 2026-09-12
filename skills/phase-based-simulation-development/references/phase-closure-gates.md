# Phase Closure Gates — God Powers / Competitive Match Logic

Use this as a concrete closure checklist after implementing a player-controlled simulation phase.

## Required sequence

1. Inspect the current dirty tree and source-of-truth PRD/balance/API rules.
2. Verify every catalog entry has an authoritative effect path in the existing tick update loop. A catalog-count test or `{ ok: true }` cast test is not roster coverage.
3. Add boundary tests for representative semantics: warning vs hazard duration, effect expiry, energy/cooldown, opening protection, ultimate unlock, target ownership, score time-up/ties, conquest, and rematch.
4. Run the producer build/typecheck before tests to avoid stale `dist` false greens:

```text
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

5. Treat every source/config/test edit after that run as invalidating the run.
6. Request an independent review against the exact current tree. A review made before the last edit is stale; do not use it as completion evidence.
7. If review finds a metadata-only or permanently-mutated derived effect, fix the authoritative tick path, rerun the full sequence, then request a fresh review.
8. Report `complete` only after the latest full suite and latest-tree review both pass. Otherwise report `verification passed; phase blocked` and name the concrete gap.

## Reusable failure patterns

- Full power roster may be metadata-complete while `applyPower` falls through to a zero-damage/no-op path. Trace every power through `applyPower` and `processPowerEffects`.
- Temporary buffs must not permanently mutate derived fields such as `maxHealth`; keep the effect in `activePowerEffects` and consume it while active so expiry is real.
- Validate the selected God against the equipped power at the server boundary, not only during match creation; do not trust a mutated/client-supplied match projection.
- Keep validation order fail-closed: status → player → equipped/God-compatible power → ban → opponent/target → energy → cooldown → target/opening rules, according to the API contract.
- Green tests prove only covered behavior. If the tests exercise 17 of 24 powers, the remaining seven need semantic coverage or the phase is not closed.
