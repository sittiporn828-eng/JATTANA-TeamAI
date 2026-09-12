# Phase 0 Verification Recipe

Use after scaffolding, from the repository root:

```bash
pnpm install
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm format:check
pnpm validate:migrations
```

For a local API smoke check:

```bash
pnpm --filter @godforge/api dev
curl -fsS http://127.0.0.1:3000/health
curl -fsS http://127.0.0.1:3000/ready
curl -fsS http://127.0.0.1:3000/api/v1
```

Then send one validated WebSocket envelope and verify a protocol response before stopping the server.

## Verified pitfalls

- A package importing another workspace package must declare the workspace dependency. Run `pnpm install` after changing manifests so pnpm creates the symlink.
- With `tsc --noEmit`, app/package typechecks may need dependency packages built first. A root `typecheck` script that runs `pnpm -r build && pnpm -r typecheck` is a simple deterministic solution for a small foundation.
- Do not let frontend `paths` map an app to another package's source when the app has a narrow `rootDir`; use the built package declarations or a proper project reference.
- `vitest` exits nonzero for packages with no tests unless `--passWithNoTests` is set. Keep at least one real deterministic test in the workspace rather than making every package pretend to have coverage.
- On Windows, raw string comparison between `import.meta.url` and `file://${process.argv[1]}` can prevent an ESM entrypoint from starting. Use `fileURLToPath(import.meta.url)` or a dedicated executable entrypoint.
- Validate local migration structure without claiming a database migration passed when no PostgreSQL/Supabase service was actually contacted.
