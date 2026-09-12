# Closure Checklist

## Before each row

- Finding has exact file/line and root cause.
- One deterministic RED reproduction exists.
- Shared callers/helper and user/timezone scope were traced.

## After each row

- Targeted regression is GREEN.
- Related tests and typecheck pass.
- Ledger and durable note record the evidence and remaining limitation.

## Final gate

- Full tests, lint, typecheck, and production build pass after the last edit.
- `git diff --check` passes and generated artifacts are restored.
- Runtime dependencies are reported separately from code evidence.
- No commit/push/deploy is claimed unless the command actually ran.
