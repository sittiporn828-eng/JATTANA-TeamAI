# JATTANA AI Coding Platform Playbook

Updated: 2026-09-12

This is the shared operating knowledge for Codex CLI, Claude Code, and Google Antigravity CLI (`agy`). It is a routing guide, not a replacement for each platform's current documentation.

## Shared rule

Use the same project truth across agents: git status, project instructions, tests, and TeamAI knowledge. Read the relevant project code before editing. Keep changes minimal, run the narrowest meaningful check, then the full project gate when practical.

Never put API keys, OAuth tokens, passwords, or production data in TeamAI docs, rules, prompts, or commits.

## Codex CLI

Official docs:

- CLI: https://developers.openai.com/codex/cli/
- Project instructions: https://developers.openai.com/codex/guides/agents-md/

Verified local CLI: `codex-cli 0.147.0`.

- Codex discovers `AGENTS.md`; use `~/.codex/AGENTS.md` for reusable user guidance and a repo-root `AGENTS.md` for project rules. Put narrower rules in the closest directory.
- Use `codex exec "..."` for bounded one-shot work and `codex review` for review.
- Prefer `--sandbox workspace-write` and explicit `-C <repo>`/`--add-dir <dir>`.
- Use `--ask-for-approval on-request` for normal work. Do not use `--dangerously-bypass-approvals-and-sandbox` unless the environment is separately controlled and the task is explicitly approved.
- Codex requires a git repository for normal coding work. Check `git status` before and after.
- Use `codex login`, `codex doctor`, and `codex update` for account/health/version maintenance.
- MCP and hooks are capability extensions; inspect and trust them before enabling them.

## Claude Code

Official docs:

- CLI: https://code.claude.com/docs/en/cli-reference
- Memory/instructions: https://code.claude.com/docs/en/memory
- Hooks: https://code.claude.com/docs/en/hooks

Verified local CLI: `claude 2.1.269` (authenticated and `claude doctor` passed).

- Claude reads `CLAUDE.md`, not `AGENTS.md` directly. To share rules without duplication, create a small `CLAUDE.md` containing `@AGENTS.md` plus Claude-specific additions.
- Keep project instructions concise; official guidance recommends targeting under 200 lines per file.
- Use `claude -p "..." --max-turns N` for bounded automation; use interactive mode for multi-turn work.
- Prefer `--allowedTools`/permission modes and explicit `--add-dir`; use hooks for enforcement, not vague prose.
- Use `claude doctor`, `claude auth status`, and `claude mcp list` for health and integration checks.
- Use worktrees for isolated feature/review work when parallel edits could conflict.

## Antigravity CLI (`agy`)

Official installer: https://antigravity.google/cli/

Verified local CLI: `agy 1.1.13`; local command reference: `agy --help`.

- Use `agy --print "..." --add-dir "<repo>"` for bounded, non-interactive tasks.
- Use `agy --output-format json` when a machine-readable result is needed.
- Use `--model` and `--effort` only when the task needs a deliberate choice; check available models with `agy models`.
- Use `--sandbox` where possible. Treat `--dangerously-skip-permissions` as exceptional, not the default.
- First use requires Google Sign-In; keep credentials in the system keyring, never in TeamAI.
- Split large work into small fixes, then run the project's build/test gate after each meaningful change.
- Always pass `--add-dir` so the workspace boundary is explicit.

## Cross-agent handoff

When switching platform, leave a short handoff in the project issue/PR or a tracked note:

```text
Task:
Files changed:
Checks run and result:
Open risk / next step:
```

Do not assume one agent's private session memory is visible to another. TeamAI docs, project instructions, git commits, and test output are the shared evidence.

## Source freshness

Re-check the official links and `codex --help`, `claude --help`, and `agy --help` after a major CLI update. This document records the last verified local versions above; it does not freeze future behavior.
