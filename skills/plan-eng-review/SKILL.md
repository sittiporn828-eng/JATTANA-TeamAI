---
name: plan-eng-review
description: "Eng manager-mode architecture review: scale limits, failure modes, data consistency, and test matrix."
version: 1.0.0
author: Garry Tan (gstack adaptation for TeamAI)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [architecture, code-review, system-design, engineering-management, reliability]
    related_skills: [plan-ceo-review, systematic-debugging, test-driven-development]
---

# Engineering Manager Architecture Review

## Overview
Rigorous technical review from a seasoned Engineering Manager. Ensures plans survive production reality: network partitions, high concurrency, data corruption, schema migrations, and race conditions.

Review only. Do not write feature code or implement during this phase.

---

## Core Engineering Principles

1. **State Machines Over Ad-Hoc Flags:**
   - Ban multiple boolean flags (`isLoading`, `isFailed`, `isRetrying`). Require explicit, exhaustive state enums or finite state machines.
2. **Defensive Data Boundaries:**
   - Never trust input across trust boundaries (APIs, Webhooks, User Input, IPC). Validate with schemas (Zod, Pydantic, TypeScript runtime guards).
3. **Idempotency Everywhere:**
   - Any mutation triggered over HTTP, queues, or webhooks must be safe to execute multiple times with the same result.
4. **Zero Catch-Alls:**
   - Never write empty `catch (e) {}` blocks. Every exception must be classified, logged with context, and either recovered or propagated.
5. **No Migration Without Rollback:**
   - Schema and lifecycle migrations must be additive and backwards-compatible with active server processes.

---

## The 5 Engineering Review Checkpoints

### 1. Architecture & Data Flow
- Map data ingress, transformation, storage, and egress.
- Identify the single source of truth for every piece of data.
- Are there hidden circular dependencies or tight coupling?
- Provide an ASCII diagram of the system flow.

### 2. Concurrency & Race Conditions
- What happens if two requests modify the same entity at the exact same millisecond?
- Are database transactions, row-level locks, or optimistic concurrency tokens in place?
- For WebSocket / realtime systems: is the state authority strictly server-side?

### 3. Failure Mode & Recovery Matrix
Enumerate every point of failure:
| Component / External Call | Failure Scenario | System Reaction | User Experience | Recovery Mechanism |
|---|---|---|---|---|
| Database / Supabase | Connection pool exhausted | Return 503 + alert | Graceful retry screen | Automatic retry with jitter |
| External API (e.g. LLM/Payment) | Timeout (30s+) | Circuit breaker trips | Informative error | Webhook reconciliation / queue retry |
| Client disconnect | Dropped midway | Rollback transaction | Re-sync on reconnect | Idempotent transaction token |

### 4. Telemetry, Observability & Gates
- What metric spikes if this feature breaks in production?
- Are structured logs emitted with correlation IDs (`trace_id`, `user_id`)?
- What health check or canary gate verifies a successful deployment?

### 5. Test Coverage & Runnable Checks
- Enforce the Ponytail rule: Every non-trivial logic branch leaves ONE fast runnable check.
- What unit test fails if the edge case occurs?
- What integration test verifies end-to-end flow?

---

## Output Format

```markdown
# Engineering Review: {Feature / System}
**Status:** [APPROVED | APPROVED WITH CONDITIONS | BLOCKED]

### 1. Architecture Assessment
[Summary of design soundness, coupling, and data boundaries]

### 2. Required Hardening Items (Before Merge)
- [ ] **Critical:** [Fix for race condition, data corruption, or silent failure]
- [ ] **Resilience:** [Timeout, retry, or idempotency requirement]
- [ ] **Observability:** [Required metric or structured log]

### 3. Test Verification Plan
- **Primary Failure Test:** `[Command or test file that reproduces failure]`
- **Happy Path Gate:** `[Command that validates green state]`
```
