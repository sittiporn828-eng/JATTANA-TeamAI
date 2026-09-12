# Godforge Phase 12 Operations Baseline

Use this checklist when starting or closing Phase 12 GM Console/LiveOps/support work:

- Read the exact Phase 12 acceptance section and split login, dashboard, live monitoring, support, bugs, player reports, moderation, compensation, redeem/store operations, feature flags, and audit logs into independent matrix rows.
- Start with one injectable operations service and test real Fastify routes through `app.inject()` before building UI.
- Player submissions derive identity from the player session. Admin mutations require a separate admin authorization boundary; a shared static bearer token is only bootstrap authentication, not GM login, session identity, RBAC, expiry, or MFA.
- If a bootstrap admin credential is retained, add a short-lived server-issued admin session and test login/logout/expiry; never call raw credential input a completed GM auth flow.
- Audit every state-changing GM action at the service boundary with actor, action, target, timestamp, and durable storage. Include existing admin economy/redeem mutations, not only newly-created OperationsService methods.
- Validate status/reason/category/severity/action unions and numeric fields at runtime before any TypeScript cast. `as never` is not validation; use an explicit allowlist first.
- Add at least one route-level regression for every sensitive route family: unauthorized admin access must fail, and an authorized mutation must persist/audit. Service-only tests do not prove the HTTP boundary.
- Do not count a static UI panel, empty monitor response, `note`-only endpoint, or shape-only endpoint as acceptance evidence. Online players, active matches, rooms, queue, player search, and match detail must be backed by real service state and traced end-to-end.
- Compensation is not complete when it only records a grant request: validate target/delivery mode and actually deliver through wallet/inventory/inbox/redeem paths, with idempotency where applicable.
- Feature flags are not complete when they can only be written/listed. Wire flag checks into the protected gameplay/shop/redeem/chat path or document the exact kill-switch consumer and test it.
- JSON snapshot persistence is acceptable only as a local vertical-slice baseline; production Phase 12 requires migration-managed durable storage and restart/concurrency evidence. Isolate test instances from production snapshots.
- Existing player-facing rooms/ranked/replay/friends APIs do not satisfy GM monitoring acceptance without admin read models and console UI.
- After the last console formatter/build/edit round, rerun full gates and dispatch a new fail-closed review. Reviewers must inspect current data wiring, persistence, auth, delivery, and UI behavior—not trust endpoint shapes or stale prior reviews.
- Report support/bug/report CRUD as `Phase 12 baseline/vertical slice` until all acceptance rows have real route/UI behavior and regression tests.
