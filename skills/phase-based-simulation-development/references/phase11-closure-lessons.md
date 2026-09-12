# Phase 11 Closure Lessons

Session-specific closure lessons for store/economy phases:

- A green suite is not enough. If review finds a schema/data-authority blocker, fix the migration and runtime path, add a regression that fails on rollback, rerun all gates, and dispatch a fresh review of the latest tree.
- A database-backed catalog is not database-authoritative when startup upserts every source-code constant. Seed defaults only when the table is empty; prove a direct persisted-row edit survives restart and is consumed by runtime.
- Normalized redeem persistence must remove the canonical full-definition JSON blob. Persist `code_type` as a constrained migration-managed field, represent all documented types in runtime/HTTP/schema, and validate type-specific invariants such as `single_use` maxUses=1 and `player_specific` requiring an owner list.
- Keep literal phase acceptance separate from broader production architecture, but do not use that distinction to excuse missing requirements explicitly documented in the phase's data model.
- Never announce completion while the required fresh fail-closed review is pending; a prior review is stale after any edit, including formatter-only edits.
