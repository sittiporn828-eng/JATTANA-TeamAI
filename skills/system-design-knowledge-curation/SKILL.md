---
name: system-design-knowledge-curation
description: "Use when curating shared system-design knowledge."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, macos, linux]
metadata:
  hermes:
    tags: [system-design, architecture, knowledge-base, repositories, provenance]
    related_skills: [understand-knowledge, github-repo-management, continuous-knowledge-capture]
---

# System Design Knowledge Curation

Turn an external architecture/system-design repository into a durable, reusable
knowledge source for a multi-project workspace. Preserve the upstream material
as a traceable reference, then add a small local layer that tells engineers and
agents when each topic applies. Do not turn an educational repo into an
unreviewed production standard.

## When to use

Use this skill when the user asks to review a system-design repository, make a
shared architecture reference, create an internal knowledge base, or route
multiple projects to common design guidance.

## Core rule

One canonical source plus one local index beats copies in every project. Keep
upstream content immutable where possible; put local interpretation in the
knowledge hub and project applicability matrix. Add per-project copies only
when the tooling demonstrably cannot reach the canonical entrypoint.

## Workflow

### 1. Inspect before designing the storage shape

- Resolve the workspace root and existing knowledge-base/vault conventions.
- Inspect the repository root, chapter/topic structure, provenance, license,
  update status, and repository size before copying it.
- Count actual topic documents from the checked-out tree; do not infer the
  count from a URL or README alone.
- Identify whether the source is instructional notes, normative standards,
  implementation code, or a mixture.

### 2. Preserve provenance

- Clone or snapshot the source into the workspace's existing reference-repo
  area, reusing its naming convention.
- Record upstream URL, exact commit hash, snapshot date, and source caveats.
- Keep the raw/reference copy separate from synthesized notes.
- If a license is absent or unclear, do not silently relicense or redistribute
  the content; retain attribution and flag the caveat.
- Keep the local copy updateable when practical (`git remote`, pinned commit).

### 3. Build the minimum useful local layer

Create only these artifacts unless the user asks for more:

1. **Knowledge hub** — purpose, source, scope, chapter index, usage steps, and
   a design gate.
2. **Project applicability matrix** — project/area → relevant topics →
   project-specific risks. Map real projects found in the workspace; do not
   invent projects.
3. **Workspace entrypoint** — short instructions discoverable by agents and
   developers in the workspace root.
4. **Change log/daily note** — record the durable result and provenance.

The design gate should cover at least scope/tenant boundaries, scale and
capacity, consistency/idempotency/ordering, failure and retry behavior,
security/auditability, and operations/rollback.

### 4. Adapt, do not cargo-cult

Every source topic is a prompt for questions, not a mandatory component. Prefer
the smallest architecture that satisfies current acceptance criteria. Do not
introduce queues, caches, sharding, service splits, or event pipelines merely
because the source describes them.

For financial, healthcare, identity, or other high-consequence domains, mark
where official documentation, threat modeling, compliance requirements, and
project-specific tests must supersede the teaching notes.

### 5. Validate the result

Before reporting completion, verify:

- the reference directory exists and points to the expected upstream;
- the pinned commit resolves;
- the topic count in the snapshot matches the generated index/matrix;
- hub, matrix, workspace entrypoint, and change log exist;
- links/paths use the workspace's actual conventions;
- the daily note records evidence, caveats, and next action;
- no secrets or environment-specific transient failures were captured.

Use the compact checklist in `references/source-evaluation-checklist.md`.

## Update workflow

When refreshing an existing knowledge source, fetch/inspect the upstream
change first, compare topic inventory and content shape, update the pinned
provenance, then revise only the local hub/matrix entries affected. Do not
rewrite the whole knowledge base or duplicate unchanged source material.

## Common pitfalls

- **Copying into every project:** creates drift and multiplies maintenance;
  use a canonical hub plus routing matrix first.
- **Treating notes as standards:** label educational/WIP material and verify
  against official docs and project constraints.
- **Unpinned references:** a moving `main` branch makes decisions
  irreproducible; record the commit hash.
- **README-only inventory:** chapter lists can be stale; count directories and
  topic documents from the snapshot.
- **Overbuilding a graph/dashboard:** a graph is optional. Add it only when
  users need graph navigation or the existing wiki format requires it.
- **Missing the workspace convention:** reuse existing `30 Resources`, `50
  Wiki`, daily notes, and index files instead of inventing parallel storage.

## Output

Report the canonical reference path, exact source commit, verified topic count,
created hub/matrix/entrypoint paths, and any caveat. Keep the report short.
