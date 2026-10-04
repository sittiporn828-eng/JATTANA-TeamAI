---
name: spec
description: "5-phase spec hardening: turn vague intent into battle-tested, implementation-ready issue/ticket."
version: 1.0.0
author: Garry Tan (gstack adaptation for TeamAI)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [specification, requirements, ticketing, task-decomposition, planning]
    related_skills: [plan-ceo-review, plan-eng-review, prd-phase-closure]
---

# 5-Phase Executable Specification

## Overview
Turns vague user ideas, Slack discussions, or bug reports into complete, unambiguous, implementation-ready technical specifications. 

**Strict Rule:** Complete all 5 phases sequentially. Do not jump to coding until Phase 5 is approved.

---

## Phase 1: Understand the "Why"
- **Root Cause Problem:** What real friction, bug, or business blocker does this address?
- **Stakeholder Context:** Why does this matter right now? What is the cost of doing nothing?
- **Quantified Impact:** Expected outcome (e.g., reduces checkout abandonment by 15%, eliminates 500 status errors on webhook retry).
- **Deduplication Check:** Search issues, PRs, and recent commits. Has this been attempted before?

## Phase 2: Scope Boundaries
- **In-Scope:** Explicit list of features, behaviors, and endpoints to build.
- **Explicitly NOT in Scope:** Features deliberately deferred to future versions to maintain focus.
- **Trade-Off Decisions:** What are we trading off? (e.g., speed of delivery vs perfection, memory usage vs compute).

## Phase 3: Technical Interrogation (Read Code First)
- Inspect existing codebase first before specifying new files.
- Grep all callers and consumers of touched functions.
- Audit data structures, database schemas, and API contracts.
- List exact files to be created, modified, or deleted.

## Phase 4: Implementation Blueprint
Break work into small, sequential phases:
- **Phase 1: Foundation & Data Schema** (Migrations, types, validation).
- **Phase 2: Core Logic & Backend** (Services, business rules, idempotency).
- **Phase 3: User Interface & Consumer Integration** (Components, state, error states).
- **Phase 4: Telemetry, Polish & Documentation** (Logs, docs, clean-up).

Every phase must include:
- Exact file paths touched.
- Acceptance criteria (Given / When / Then).
- ONE runnable verification command (test, lint, or smoke script).

## Phase 5: Quality Gate & Final Specification Document

Assemble the final backlog-ready Markdown specification:

```markdown
# Spec: {Feature Title}

## 1. Problem Statement & Context
{Why this matters, who it impacts, and current pain}

## 2. Verified Current State
- Existing files touched: `{paths}`
- Shared callers audited: `{callers}`
- Known constraints: `{constraints}`

## 3. Scope
- **Included:** {bulleted list}
- **Excluded (Out of Scope):** {bulleted list}

## 4. Implementation Plan
### Phase 1: {Foundation}
- Files: `{files}`
- Changes: {summary}
- Verification: `{command}`

### Phase 2: {Core Logic}
...

## 5. Acceptance Criteria
- [ ] Given {state}, when {action}, then {expected outcome}
- [ ] No regression on {sibling components}
- [ ] Zero silent error handling
```
