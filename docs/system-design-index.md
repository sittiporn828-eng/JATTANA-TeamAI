# JATTANA System Design Knowledge

Shared architecture reference for JATTANA GROUP projects.

## Source

- Upstream: https://github.com/liquidslr/system-design
- Pinned snapshot used locally: `9d8388721e7231442763ad37398b8d82224aa68f`
- The upstream notes are work in progress and educational material. Verify production, security, healthcare, and financial decisions against primary documentation and project evidence.

## Use before design work

1. Choose relevant chapters in `project-applicability.md`.
2. State scope, tenant boundary, scale, consistency, idempotency, failure behavior, security, and operations.
3. Prefer the smallest design that satisfies the measured requirement.
4. Record the decision and evidence in the project repository.

## Design gate

- Scope and data ownership are explicit.
- Peak RPS, payload, concurrency, storage growth, and retention are estimated.
- Consistency, ordering, idempotency, and transaction boundaries are defined.
- Timeout, retry, duplicate, partial failure, backpressure, and recovery are defined.
- Authn/authz, tenant isolation, secrets, auditability, and abuse limits are covered.
- Logs, metrics, traces, alerts, migration, and rollback are testable.

See `project-applicability.md` for routing by project.
