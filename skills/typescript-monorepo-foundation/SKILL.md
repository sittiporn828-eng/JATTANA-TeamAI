---
name: typescript-monorepo-foundation
description: "Use when bootstrapping a TypeScript pnpm monorepo."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [typescript, pnpm, monorepo, fastify, websocket, supabase, ci, foundation]
---

# TypeScript Monorepo Foundation

Build the smallest runnable engineering foundation before feature work. Treat the project's design/architecture documents as the source of truth, inspect the existing tree before creating files, and stop at the requested phase.

## Scope

Use for a new or incomplete TypeScript game/backend monorepo containing multiple apps and shared packages. This skill covers repository foundation, not gameplay, product features, or production infrastructure scale-out.

Expected baseline:

```text
apps/
  web-game/
  gm-console/
  api/
  game-server/
packages/
  simulation/
  game-config/
  network-protocol/
  database/
  shared/
  ui/
  testing/
docs/
supabase/migrations/
infrastructure/docker/
.github/workflows/
```

## Workflow

1. **Read source-of-truth first.** Locate the required docs, read all of them, and record locked architecture, current phase, package manager, runtime versions, endpoint contracts, database tables, balance/config rules, UI tokens, and operational constraints. If docs are at the repository root but the requested canonical location is `docs/`, move them without rewriting their contents.
2. **Inspect before scaffolding.** Check repository status, remotes, existing package manifests, workspace files, app/package directories, configs, migrations, and CI. Reuse existing files; do not overwrite an implementation that is already present.
3. **Lock the toolchain.** Add `packageManager`, `engines`, `.nvmrc`/equivalent, `pnpm-workspace.yaml`, root TypeScript config, ESLint, Prettier, `.env.example`, and `.gitignore`. Never place real secrets in tracked files.
4. **Create package boundaries.** Each app/package gets its own manifest, scripts, and tsconfig. Keep simulation independent of React, PixiJS, cloud SDKs, network clients, and database clients. Keep the game server authoritative: client messages are intents, not trusted match/rank/wallet/inventory state.
5. **Implement one thin vertical tracer.** Add only the minimum useful behavior: API `/health`, `/ready`, `/api/v1`; a WebSocket message envelope with runtime validation; a versioned config object; a seeded RNG and deterministic simulation step; a structured logger; a database idempotency primitive; UI tokens/locales; and one deterministic test.
6. **Add the persistence baseline.** Create an ordered, transactional SQL migration for the foundation tables required by the docs. Include balance-config version/checksum, feature flags, audit logs, player profiles, and idempotency keys when required by the architecture. Add a deterministic migration validation script even when no external database is available.
7. **Add repeatable quality gates.** Root scripts must cover install, lint, typecheck, test, build, formatting, and migration validation. Packages without tests may use `--passWithNoTests`, but at least one real foundation test must exist. CI must run the same gates with the locked Node/pnpm versions.
8. **Smoke test the runtime.** Start the API, request `/health`, `/ready`, and `/api/v1`, then send one valid WebSocket message and verify the response. Stop the process after the check. On Windows ESM entrypoints, do not compare `import.meta.url` to a raw Windows `process.argv[1]` string; use `fileURLToPath(import.meta.url)` or a simple dedicated entrypoint.
9. **Verify external boundaries honestly.** If Docker, PostgreSQL/Supabase, or GitHub Actions cannot be exercised, report those checks as blocked. Do not fabricate a passing external verification. Local lint/typecheck/test/build/migration checks remain useful but do not replace service-backed verification.
10. **Stop at the phase boundary.** Do not add world simulation, civilization, war, powers, matchmaking, ranked systems, commerce, or GM features when the request is Phase 0.

## TypeScript resolution rules

- Workspace dependencies must be declared in each consuming package.
- Prefer package `main`/`types` pointing at `dist` for app-to-package boundaries.
- Build dependency packages before app typechecks when using `--noEmit`; a root `typecheck` can run the build first, then recursive typechecks.
- If a source path mapping is necessary, make it relative to the package's own tsconfig and keep the mapping local to that package. Do not make every app compile another package's source under its own `rootDir`.
- Run `pnpm install` after changing workspace dependencies so symlinks and the lockfile are current.

## Security and authority checks

- Validate all untrusted HTTP/WebSocket payloads at the boundary.
- Never trust client claims for match outcome, rank/MMR, wallet balance, purchases, inventory, or administrative actions.
- Keep balance values in versioned game config, not scattered literals or environment variables.
- Use transaction/idempotency foundations for retryable economy mutations.
- Keep secrets only in environment/service configuration; `.env.example` contains names and safe defaults only.

## Acceptance checklist

- [ ] Required docs read and retained as Source of Truth
- [ ] Existing repo inspected before changes
- [ ] Four apps and six required shared packages exist
- [ ] Node/pnpm versions locked and lockfile generated
- [ ] TypeScript, lint, format, test, and build commands are runnable
- [ ] Environment schema rejects invalid trusted configuration
- [ ] API health/readiness/versioned route smoke-tested
- [ ] WebSocket envelope and runtime validation smoke-tested
- [ ] Transactional migration validates locally
- [ ] Balance config, feature flags, audit, profiles, and idempotency baselines exist
- [ ] Seeded deterministic test passes
- [ ] UI tokens and locales exist
- [ ] Structured logging exists
- [ ] Dockerfiles and CI workflow are present
- [ ] No Phase 1 feature code was added
- [ ] External blockers are explicitly reported

## Supporting reference

- `references/phase0-verification-recipe.md` — concise command sequence and the verified Windows/TypeScript pitfalls from the foundation bootstrap workflow.
