---
name: production-hardening-review
description: "Use when hardening harnesses for production."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [production, hardening, security, recovery, structured-output, patches]
---

# Production Hardening Review

Use this skill when a model-driven harness, workflow runner, patch applicator, or automation wrapper must move from foundation/pre-release to production. Keep the wrapper thin: harden boundaries and recovery; do not duplicate the underlying runtime's provider, tools, sessions, or memory.

## Release gate

Do not label Production V1 until all Critical/High findings are closed, the real runtime path has executed, and the complete verification suite passes after the final edit. If the repository is not Git, review changed files directly and record compile/tests/mutation evidence instead of inventing a diff.

## Workflow

1. Establish baseline: inspect the real flow end-to-end, run the existing tests, compile/lint where available, and record the environment. Do not trust a prior test count after later edits.
2. Write failing tests for each hardening requirement before implementation: malformed output, runtime timeout/crash, bad session ID, corrupted state, traversal, unexpected mutation, interrupted apply, duplicate resume, missing workspace, and non-git workspace.
3. Enforce structured output semantically. Parse only a strict JSON contract with required fields and deterministic delimiters. Clean terminal ANSI output first. A parseable `status: blocked|failed` remains blocked/failed; valid JSON alone is not success. Retry malformed output, then fail closed.
4. Test real runtime behavior: one real invocation, session extraction from actual output formats, one real resume, and provider failure/iteration exhaustion. Record usage as `unavailable` when no provider payload exists; never estimate actual tokens.
5. Isolate all model analysis and verification in a disposable workspace. Reject symlinks or copy without following them, exclude control/build directories, detect source mutation, validate workspace binding on resume, and reject path traversal.
6. Make fixes patch-based: audit/report → plan → generate diff → validate boundaries/stale hashes → show changed files → require explicit per-file allowlist and exact patch hash → atomic apply → backup → test → review diff → finalize. No blind source-file copy.
7. Make state crash-consistent: write report/index/trace artifacts before terminal `reported`; use atomic writes; kill/restart at exploration, audit, verification, and report; assert same run ID and idempotent duplicate resume. Add locking/CAS before claiming concurrent resume safety.
8. Enforce budgets on the real path. Wall-clock and max-turn limits must reach subprocess execution. If a token budget is requested but provider usage is missing or exceeds the limit, fail closed instead of fabricating usage.
9. Dispatch an independent read-only reviewer with the actual changed files plus test evidence. Treat Critical/High findings as release blockers. Re-run the full suite after every fix cycle.
10. Write a release report listing evidence, unresolved risks, exact blockers, and the final decision.

## Windows requirements

Use Windows-aware parsing for configured executable paths with spaces and actually wire the configured command. Avoid shell execution. Validate positive timeout values and catch `TimeoutExpired`, `OSError`, and invalid command errors.

## Review pitfalls

- Executable-name allowlists are not sandboxes: `python -c`, npm scripts, and git hooks can mutate files. Use disposable verification workspaces.
- Approval without a patch hash approves a new, potentially different model output. Bind approval to the exact preview hash.
- Marking a run `reported` before writing its report loses artifacts after a crash.
- A unit fake can hide ANSI wrappers, echoed prompts, upstream 503s, and real session/resume behavior.
- A clean test run before the last edit is not final evidence.

## References

See `references/production-hardening-review.md` for the reusable adversarial matrix and evidence checklist.
