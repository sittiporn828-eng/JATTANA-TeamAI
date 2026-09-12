# Catalog-complete Sandbox Reference

For a request to expose every implemented PRD/catalog capability in a local sandbox:

1. Read the authoritative simulation `powerCatalog`; never maintain a second UI-only power list.
2. Add a distinct `createSandboxMatch()` path that loads the full catalog and marks the match sandbox-only.
3. Keep `createCompetitiveMatch()` and ranked loadout limits unchanged.
4. Scope sandbox relaxations explicitly to local experimentation: god/ranked/energy/cooldown/ultimate gates may be bypassed only when `match.sandbox` is true. Preserve target validation and authoritative state mutation.
5. Add a regression test proving Sandbox accepts the full catalog while Competitive rejects an oversized loadout.
6. Browser-check every category and inspect the console before claiming closure.

This pattern prevents a complete catalog from crashing the UI by being passed into a ranked constructor, while keeping competitive contracts fail-closed.