# Project Applicability

Starting points, not mandatory architecture.

| Project | Start with | Main risks |
|---|---|---|
| JATTANA OS | scaling, estimation, framework, key-value, IDs, queues, monitoring, payment, wallet | tenant isolation, audit, approval idempotency, financial consistency |
| MediLINE | scaling, framework, key-value, IDs, notifications, queues, monitoring, email, object storage | patient privacy, access control, immutable audit trail |
| Godforge | scaling, estimation, framework, IDs, queues, monitoring, leaderboard | authoritative 20TPS state, ordering, reconnect, backpressure |
| Meekamrai | framework, key-value, IDs, notifications, queues, monitoring, reservation, payment, wallet | inventory/recipe consistency, retries, branch boundaries |
| CareMate | framework, key-value, IDs, notifications, monitoring, reservation, email, object storage | booking conflicts, personal data, document access |
| Grider | scaling, framework, IDs, proximity, nearby friends, maps, queues, monitoring | geo accuracy, stale location, offline/retry, location privacy |
| Cardstaff | scaling, estimation, framework, IDs, queues, monitoring, leaderboard | realtime state, ranking integrity, abuse prevention |

Use the matching chapter files under `chapters/` only as design input; do not copy their architecture without project-specific evidence.
