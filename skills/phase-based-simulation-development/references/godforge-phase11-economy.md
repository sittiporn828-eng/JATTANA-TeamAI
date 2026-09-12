# Godforge Phase 11: economy, store, redeem, and payments

## Minimal architecture

Keep one injectable `EconomyService` responsible for wallet, append-only ledger, inventory ownership, product catalog, redeem-code state, purchases, and provider receipts. Reuse API service injection and test domain behavior plus `app.inject()` boundaries.

## Trust boundaries

- Wallet credits, redeem-code definition, and payment callbacks are server/admin/provider-only.
- A client-facing `payment_success` payload must always be rejected. Provider callback verification should be injected as `paymentVerifier`; default it to fail closed.
- Derive player, region, level, rank, and season from authenticated server state, never from request body/query.
- Validate every redeem field at the HTTP boundary. Reject malformed config rather than silently dropping eligibility fields.

## Durable model: normalized tables, not an application snapshot

Use normalized SQLite tables for `wallet`, immutable `ledger`, `inventory`, `purchases`, `redeem_codes`, `redemptions`, and `payments`/receipts. Do **not** serialize all economy Maps into one JSON state row: separate service instances load stale snapshots and overwrite each other's wallets, inventory, ledger, and limits.

Every monetary flow must use one `BEGIN IMMEDIATE` transaction:

1. Verify product/code/provider facts.
2. Read and enforce balance, account limit, ownership, and code limit from DB.
3. Insert the unique reservation/receipt (`(code, player)`, provider transaction ID, purchase row).
4. Update wallet, append ledger, grant inventory, and write receipt.
5. Commit once.

Never commit a redeem/payment claim separately from the grant: a crash between those writes permanently consumes the code or callback without delivering the reward.

Use DB constraints for provider transaction IDs, `(redeem_code, player)`, ownership `(player, item)`, and product account counts. A `Map`/`Set` is not a substitute for a shared database constraint.

On Node 22, built-in `node:sqlite` can provide this without a dependency. Vite/Vitest may not resolve a direct `node:sqlite` import; use `createRequire(import.meta.url)('node:sqlite')`. Close file-backed test DBs before deleting them on Windows.

## Product and rewards

Products are data-driven: ID, SKU, category, price, region, active/time window, account limit, discount, and rewards. Enforce every field rather than merely declaring it. Inventory is server-owned. Cosmetic/season-pass rewards must be explicitly non-competitive and must never feed ranked/Elo/match simulation inputs.

Define and test deterministic partial-bundle ownership handling. Model both season-pass tracks: free-track entitlement/claim must work without a price, premium is a purchase. Persist and return a provider receipt containing transaction, player, product, reward, and status.

## Redeem code validation checklist

Validate exists, active, start/end time, positive integer max uses, per-account use, player targeting, region, level, rank, season, and event eligibility before grant. Validate arrays and numeric fields before persisting the code. Reserve/increment use and grant inside the same transaction so simultaneous redemptions cannot exceed the limit.

## User-visible execution evidence

For long economy hardening, report only outputs that prove work happened: a focused test command with its result, the current `git status --short`, a full-gates result, or an independent review verdict. Do not claim work continues between chat turns without a verified tracked process. If the user asks to see progress, run one bounded verification command and show its unabridged relevant output.

## Closure tests

1. Restart preserves wallet, inventory, ledger, code, purchase count, redemption, and receipt.
2. Two service instances sharing one DB cannot lose concurrent wallet writes or double-purchase/overrun an account limit.
3. Duplicate provider callback grants once and returns the same persisted receipt.
4. Client payment completion is denied even when it says success.
5. Redeem max-use and same-account redemption are race-safe/idempotent; each eligibility predicate and malformed admin config has an HTTP test.
6. Store routes ignore spoofed region; free/premium pass, region/time windows, partial bundle ownership, discount, and limit paths have tests.
