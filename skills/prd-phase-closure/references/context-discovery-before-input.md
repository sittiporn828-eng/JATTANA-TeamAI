# Context discovery before requesting user input

Use this when a phase audit appears blocked by missing requirements, provider details, schemas, or design decisions.

## Order of operations

1. Read the exact PRD phase and adjacent architecture docs in the active repo.
2. Search repo docs, ADRs, migrations, `.env.example` (never print secret values), and source/tests for the allegedly missing term.
3. Search known secondary project roots only (for Windows, inspect `D:/Users/<user>/Documents/GitHub`, `D:/Users/<user>/Desktop`, and explicitly named project folders). Avoid scanning `$Recycle.Bin`, `Windows`, SDKs, binaries, `node_modules`, build output, and caches.
4. Search session history for the user's earlier artifact or decision. Session history is secondary context; verify any claimed current file/config in the original source.
5. Report found evidence with paths/sections before asking anything.
6. Ask only for the minimum missing contract. For payments this is normally the provider identity and provider-specific webhook/receipt contract—not secrets, credentials, or generic PRD requirements already present.

## Classification

- **Already specified:** implement from the repo/PRD; do not ask the user to resend it.
- **Specified generically but provider-specific detail missing:** ask for provider name/contract only; public documentation can then be researched.
- **Secret/config missing:** ask for a safe non-secret setup path or environment-variable name; never ask the user to paste secrets.
- **Design genuinely unspecified:** ask a focused decision question with the smallest number of options.

## Godforge Phase 11 evidence pattern

The Phase 11 PRD defines the payment flow, redeem code types, product fields, and cosmetic fairness. `docs/API.md` defines webhook signature/raw-body/audit requirements and redeem admin routes. `docs/DATABASE.md` defines normalized `redeem_codes`/`redeem_history` fields and code types. Read the exact Phase 11 section and the phase-specific database note before treating broader ADR architecture as a phase blocker: the current Phase 11 database note intentionally scopes the economy adapter to SQLite and describes PostgreSQL as a later adapter. Do not add speculative production infrastructure solely to satisfy that later target.

The implementation may still need a provider-specific identity/contract. A generic verifier callback is not proof of provider verification. Ask for only the provider name or redacted public callback shape if it cannot be found in the repo or history.

## Pitfall

Do not ask for missing information before searching. The user explicitly corrected this workflow: they had already supplied the requirements, and the correct action was to inspect the repo, the D: project roots, and conversation history first. For provider work, identify the provider before inferring a platform from unrelated account/profile context; after the provider is known, use official public docs for the signed webhook contract and ask only for safe runtime variable names, never secrets.

## Verified Stripe implementation pattern

For Stripe, preserve raw JSON before parsing, verify `Stripe-Signature` with `STRIPE_WEBHOOK_SECRET` and a timestamp tolerance, choose one canonical grant event (`payment_intent.succeeded` avoids cross-event double grants), require a complete per-SKU mapping of `priceId`, minor-unit `amount`, and ISO `currency`, and compare those values against the signed PaymentIntent. Use the PaymentIntent ID as the idempotency key. Keep runtime setup deferred only when the production webhook secret, price map, or Checkout/PaymentIntent creation is intentionally not configured; do not call that “provider verification missing” once the signed boundary is implemented.
