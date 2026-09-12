---
name: jattana-agent-workflow
description: Use when coding in JATTANA GROUP with Codex, Claude Code, or Antigravity. Apply shared rules and verify changes.
---

# JATTANA Agent Workflow

1. Read the project instructions and `teamai recall` relevant terms before editing.
2. Check `git status`; do not overwrite unrelated work.
3. Trace callers and the real flow before changing shared code.
4. Make the smallest working change; preserve security, validation, error handling, and accessibility.
5. Run the narrowest relevant check, then the project's full gate when practical.
6. Report changed files, exact checks, failures, and remaining risk.

## Platform routing

- Codex: use `AGENTS.md`, `codex exec`, `--sandbox workspace-write`, and explicit approval boundaries.
- Claude Code: use `CLAUDE.md`/`.claude/rules`, `claude -p`, bounded turns, and hooks for enforcement.
- Antigravity: use `agy --print`, explicit `--add-dir`, optional `--sandbox`, and split large tasks.

## Shared knowledge

For architecture questions, search TeamAI before designing:

```bash
teamai recall "<feature, failure mode, or project>"
```

Use `docs/system-design-*.md` as reference only. Confirm project-specific scale, consistency, security, failure behavior, and operational cost before implementation.
