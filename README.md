# JATTANA TeamAI

Shared TeamAI harness for JATTANA GROUP AI agents.

## Contents

- `docs/system-design-*.md` — shared system design knowledge and 28 reference chapters
- `teamai.yaml` — TeamAI configuration
- `learnings/` — durable team learnings created through reviewed workflow
- `skills/<name>/SKILL.md` — 106 agent skills; the full catalog is in [`SKILL.md`](SKILL.md)

## Recently added skills

| Skill | Use for | Source |
|---|---|---|
| `humanizer` | Rewrite AI-sounding text so it reads like a person wrote it | [blader/humanizer](https://github.com/blader/humanizer) |
| `hallmark` | Anti-AI-slop page design, audits, redesigns, design extraction | [Nutlope/hallmark](https://github.com/Nutlope/hallmark) |
| `impeccable` | Design, critique, audit, polish, and harden frontend interfaces | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) |
| `ui-ux-pro-max` | UI/UX design intelligence: styles, palettes, fonts, charts, stack guidance | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) |
| `banner-design` | Social, ad, hero, and print banners | same as above |
| `brand` | Brand voice, visual identity, messaging, consistency | same as above |
| `design` | Brand identity, tokens, logo/icon/CIP generation, HTML slides | same as above |
| `design-system` | Primitive→semantic→component tokens and component specs | same as above |
| `slides` | Strategic HTML presentations with Chart.js | same as above |
| `ui-styling` | shadcn/ui, Tailwind CSS, canvas-based visuals | same as above |
| `ponytail` | Smallest change that fully solves the task (YAGNI, stdlib first) | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) |
| `ponytail-review` | Quality review of a change | same as above |
| `ponytail-audit` | Whole-repo quality audit (read-only report) | same as above |
| `ponytail-debt` | Ledger of `shortcut:` comments | same as above |
| `ponytail-gain` | Measured savings from the ponytail benchmark | same as above |
| `ponytail-help` | Ponytail levels, skills, and commands | same as above |
| `scrapling-official` | Scrape/crawl sites with anti-bot bypass and stealth browsing | [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) |
| `diagram-design` | Branded HTML/SVG diagrams (updated to latest upstream) | [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) |
| `office-hours` | YC office-hours diagnostic and brainstorming | Garry Tan gstack |
| `plan-ceo-review` | CEO/founder-mode plan review | Garry Tan gstack |
| `plan-eng-review` | Eng-manager architecture review | Garry Tan gstack |
| `spec` | Turn vague intent into an implementation-ready ticket | Garry Tan gstack |

Before using: `scrapling-official` needs `pip install scrapling`, and logo/icon generation in `design` needs `GEMINI_API_KEY` set on your machine (never in this repo).

## Local setup

```bash
npm install -g teamai-cli
teamai init https://github.com/sittiporn828-eng/JATTANA-TeamAI --scope user
teamai recall enable
teamai pull
teamai status
```

The repository is private. Do not store API keys, passwords, tokens, or production secrets here.
