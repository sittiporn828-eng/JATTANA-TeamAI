# JATTANA TeamAI

Shared TeamAI harness for JATTANA GROUP AI agents.

## Contents

- `docs/system-design-*.md` — shared system design knowledge and 28 reference chapters
- `teamai.yaml` — TeamAI configuration
- `learnings/` — durable team learnings created through reviewed workflow

## Local setup

```bash
npm install -g teamai-cli
teamai init https://github.com/sittiporn828-eng/JATTANA-TeamAI --scope user
teamai recall enable
teamai pull
teamai status
```

The repository is private. Do not store API keys, passwords, tokens, or production secrets here.
