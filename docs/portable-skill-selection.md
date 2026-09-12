# Portable skill selection

Updated: 2026-09-12

## Audit result

- Hermes installed skills scanned: 179
- Codex native skills inspected: `.codex/skills/.system/imagegen` and `.codex/skills/.system/openai-docs`; these are OpenAI-specific and stay native
- Claude local skills inspected: `jattana-agent-workflow`, `teamai-share-learnings`, `team-wiki-codebase`; the TeamAI skills are already distributed, while `team-wiki-codebase` remains Claude-oriented
- Antigravity: no separate local skill directory was found under its Windows CLI/app paths; its portable interface is the CLI prompt/flags (`--print`, `--add-dir`, `--sandbox`)

## Added to TeamAI

These skills are useful across Codex, Claude Code, and Antigravity because they are instruction-first and do not require Hermes tool APIs:

- `baseline-ui`
- `fixing-accessibility`
- `fixing-motion-performance`
- `improve-ui`
- `open-code-review-delegate` — requires the optional `ocr` CLI; the skill itself does not require a specific AI platform
- `jattana-agent-workflow` — existing JATTANA shared workflow

## Deliberately excluded

- Hermes-only skills: depend on Hermes tools, vault helpers, Telegram, or Hermes delegation APIs
- Platform-native skills: Codex `imagegen`/`openai-docs`, Claude-only `team-wiki-codebase`
- Service integrations: credentials or a provider-specific CLI are required
- Project-specific skills: Godforge/MediLINE/Meekamrai workflows should be enabled only for the matching project, not every agent
- Antigravity-specific instructions: kept in `docs/agent-platforms.md` because no portable skill package was found locally

## Rule

A skill is portable only when another agent can follow it from plain instructions, project files, and ordinary shell/git behavior. A skill that names Hermes-only tools or assumes a specific provider stays local to that platform.
