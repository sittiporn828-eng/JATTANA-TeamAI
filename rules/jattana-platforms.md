# JATTANA platform rules

- Use the shared TeamAI knowledge before architecture changes; do not copy a design blindly.
- Keep platform instructions in the correct native file: `AGENTS.md` for Codex, `CLAUDE.md` for Claude Code, and explicit `agy` flags for Antigravity.
- Use bounded execution and explicit workspace boundaries.
- Never store secrets in shared docs, skills, rules, prompts, or commits.
- Verify changes with real project checks and report evidence.
