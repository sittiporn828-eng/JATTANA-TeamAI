---
name: jattana-teamai
description: Complete JATTANA TeamAI knowledge base, engineering methodology (Ponytail), platform rules, and 85 specialist skills catalog for full-stack engineering, AI automation, UI/UX, and ops.
---

# JATTANA TeamAI — Master Skill

This master skill consolidates the entire JATTANA GROUP engineering standards, multi-agent workflows, platform rules, and domain capabilities.

## 1. JATTANA Platform Rules & Core Philosophy

- Use the shared TeamAI knowledge before architecture changes: Inspect existing helpers, schemas, and patterns before writing new ones.
- Native platform instruction files: AGENTS.md for Codex, CLAUDE.md for Claude Code, explicit flags for Antigravity (agy).
- Model Hierarchy: Prioritize claude-opus-5.5 when quota is available; fall back to gpt-6-sol (or gpt-6-luna for lightweight design) via Codex.
- Bounded execution: Work within explicit workspace boundaries. Never touch files outside task scope.
- No secrets in code or repos: Never hardcode credentials, API tokens, private keys, or passwords.
- Real project verification: Always verify changes with real project checks, linters, or test gates before reporting.

## 2. Ponytail — Efficient Engineering Methodology

1. The Ladder:
   - 1) Does this need to exist? (YAGNI)
   - 2) Already in this codebase? Reuse it.
   - 3) Stdlib does it? Use it.
   - 4) Native platform feature covers it? Use it (CSS over JS, DB constraint over app code).
   - 5) Installed dependency solves it? Use it. Never add new packages for what a few lines can do.
   - 6) Can it be one line? One line.
   - 7) Minimum code that works.
2. Inspect first: Read the code, trace the flow end-to-end before editing.
3. Fix root causes once: Check every caller of shared functions; do not patch only the symptom.
4. Runnable checks: Leave one runnable check (an assert or minimal test) for non-trivial logic.
5. Output format: Shortest diff, shortest explanation.

## 3. TeamAI 85-Skill Catalog & Routing Directory

### analytics
- Description: "Set up, improve, or audit analytics tracking: GA4, GTM, UTM."
- File Path: skills/analytics/SKILL.md

### animate
- Description: Build an animation from scratch, making the decisions in the order that determines whether it feels right — should it animate at all, what purpose, which tool, which properties, which curve and duration, how it interrupts, how it exits. Writes the implementation. Use when asked to animate something, add motion, make a component feel alive, or build a transition. For critiquing existing motion use review-animations; for auditing a whole codebase use improve-animations.
- File Path: skills/animate/SKILL.md

### animation-vocabulary
- Description: Reverse-lookup glossary that turns a vague description of a web animation or motion effect into its exact term ("the bouncy thing when a popover opens" → Pop in; "the iOS rubber-band scroll" → Rubber-banding). Use when the user asks "what's it called when…", or describes a motion effect without knowing its name and wants the right word to prompt an AI or designer with. For naming an effect, not designing or building one.
- File Path: skills/animation-vocabulary/SKILL.md

### apple-design
- Description: Apple's approach to interface design and fluid, physical motion, translated for the web. Use when building or reviewing gesture-driven UI, spring animations, drag/swipe/sheet interactions, momentum and interruptible transitions, translucent materials and depth, typography (optical sizing, tracking, leading), reduced-motion, or the design foundations (feedback, spatial consistency, restraint) behind Apple-style interfaces.
- File Path: skills/apple-design/SKILL.md

### arxiv
- Description: "Search arXiv papers by keyword, author, category, or ID."
- File Path: skills/arxiv/SKILL.md

### ask-sonner
- Description: Guide to Sonner, the React toast library — install and wire up the Toaster, pick the right toast() call, promise and loading toasts, updating, dismissing and persisting toasts, styling, theming and icons, positioning and multiple toasters. Use when working with Sonner or troubleshooting it — toasts that don't appear, appear twice, lose their styles, ignore Tailwind classes, sit behind a modal, or don't follow dark mode.
- File Path: skills/ask-sonner/SKILL.md

### backup-verification
- Description: "Use when verifying backups: checksums, gzip/tar integrity."
- File Path: skills/backup-verification/SKILL.md

### baseline-ui
- Description: Quickly deslop UI code by fixing spacing, hierarchy, typography, and small layout issues. Use when the interface needs a fast cleanup or polish pass.
- File Path: skills/baseline-ui/SKILL.md

### browser-game-performance-profiling
- Description: "Measure and fix real browser-game frame performance."
- File Path: skills/browser-game-performance-profiling/SKILL.md

### bullmq-redis-inspection
- Description: "BullMQ queues via Redis: failed jobs, opts, retry fixes."
- File Path: skills/bullmq-redis-inspection/SKILL.md

### ci-reliability-debugging
- Description: "Use when CI exposes flaky async tests; fix races remotely."
- File Path: skills/ci-reliability-debugging/SKILL.md

### claude-design
- Description: Design one-off HTML artifacts (landing, deck, prototype).
- File Path: skills/claude-design/SKILL.md

### codebase-inspection
- Description: "Inspect codebases w/ pygount: LOC, languages, ratios."
- File Path: skills/codebase-inspection/SKILL.md

### competitor-profiling
- Description: "Research and profile competitors from URLs into structured dossiers."
- File Path: skills/competitor-profiling/SKILL.md

### content-strategy
- Description: "Plan content strategy: topics, pillars, editorial calendar, roadmap."
- File Path: skills/content-strategy/SKILL.md

### copywriting
- Description: "Write, rewrite, or improve persuasive marketing copy for any page."
- File Path: skills/copywriting/SKILL.md

### customer-research
- Description: "Conduct and analyze customer research: ICP, personas, JTBD, reviews."
- File Path: skills/customer-research/SKILL.md

### diagram-design
- Description: Create branded architecture, IT current-state, flowchart, sequence, state machine, ER/data model, timeline, swimlane, quadrant, radar/spider, polar chart (polar/radial lollipop), loop/flywheel, nested, tree, org chart, layer stack, Venn, pyramid/funnel, treemap, bar, waterfall, line, Gantt and scatter charts, high-level, process, medallion, data flow, DP integration, DP security matrix, Sankey, fishbone, Wardley map, kanban, user journey, deployment, dependency graph, UML class, story map, or database schema diagrams as standalone HTML/SVG/PNG. Redraw .drawio/.drawio.png/.drawio.svg, Mermaid .mmd, or Excalidraw .excalidraw sources at a chosen size/detail; onboard brand tokens from a website; add semantic patterns, callouts, accessible motion, or sketchy/hand-drawn styling.
- File Path: skills/diagram-design/SKILL.md

### document-to-action-items
- Description: "Extract cited obligations, deadlines, tasks from documents."
- File Path: skills/document-to-action-items/SKILL.md

### docx
- Description: Create, read, edit, template, and review Word .docx files.
- File Path: skills/docx/SKILL.md

### doggy-vault-search
- Description: Use when answering questions about prior JATTANA work, decisions, projects, or user preferences. Search the Doggy knowledge vault before answering.
- File Path: skills/doggy-vault-search/SKILL.md

### e2e-encrypted-local-first
- Description: Use for E2E zero-knowledge sync / client-side WebCrypto.
- File Path: skills/e2e-encrypted-local-first/SKILL.md

### emil-design-eng
- Description: This skill encodes Emil Kowalski's philosophy on UI polish, component design, animation decisions, and the invisible details that make software feel great.
- File Path: skills/emil-design-eng/SKILL.md

### find-animation-opportunities
- Description: Search a codebase or UI for places that don't animate but should, and reject everything that shouldn't. Read-only; it proposes motion with exact values, it does not implement it. Use when the user asks "what could be animated here?" or wants to "make this feel more alive". For fixing existing animations, use improve-animations or review-animations instead.
- File Path: skills/find-animation-opportunities/SKILL.md

### fixing-accessibility
- Description: Audit and fix HTML accessibility issues including ARIA labels, keyboard navigation, focus management, color contrast, and form errors. Use when adding interactive controls, forms, dialogs, or reviewing WCAG compliance.
- File Path: skills/fixing-accessibility/SKILL.md

### fixing-motion-performance
- Description: Audit and fix animation performance issues including layout thrashing, compositor properties, scroll-linked motion, and blur effects. Use when animations stutter, transitions jank, or reviewing CSS/JS animation performance.
- File Path: skills/fixing-motion-performance/SKILL.md

### full-stack-flow-closure
- Description: "Use when verifying every flow across a multi-app system."
- File Path: skills/full-stack-flow-closure/SKILL.md

### game-prd-hardening-closure
- Description: "Use when closing a game PRD after hardening edits."
- File Path: skills/game-prd-hardening-closure/SKILL.md

### game-ui-polish
- Description: Use when polishing game UIs.
- File Path: skills/game-ui-polish/SKILL.md

### gemini-api-dev
- Description: Use this skill when writing code that calls the Gemini API for text generation, multi-turn chat, multimodal understanding, image generation, video generation, streaming responses, background research tasks, function calling, structured output, or migrating from the old generateContent API. Covers SDK usage and best practices for Gemini models and agents in Python and TypeScript.
- File Path: skills/gemini-api-dev/SKILL.md

### gemini-live-api-dev
- Description: Use this skill when building real-time, bidirectional streaming applications with the Gemini Live API. Covers WebSocket-based audio/video/text streaming, voice activity detection (VAD), native audio features, function calling, session management, ephemeral tokens for client-side auth, live translation, and all Live API configuration options. SDKs covered - google-genai (Python), @google/genai (JavaScript/TypeScript).
- File Path: skills/gemini-live-api-dev/SKILL.md

### gemini-omni-flash-api
- Description: Use this skill for generative video editing, text-to-video, image-referenced video generation, first-frame-to-video, first-and-last-frame transitions, and video extensions using Gemini Omni 1.1 Flash (gemini-omni-1.1-flash) via the official google-genai SDK. Includes workflows for pre-processing/optimizing high-resolution or long source videos with ffmpeg, stripping audio for full sound regeneration, and handling turn-by-turn video editing and parallel execution.
- File Path: skills/gemini-omni-flash-api/SKILL.md

### github-auth
- Description: "GitHub auth setup: HTTPS tokens, SSH keys, gh CLI login."
- File Path: skills/github-auth/SKILL.md

### github-code-review
- Description: "Review PRs: diffs, inline comments via gh or REST."
- File Path: skills/github-code-review/SKILL.md

### github-issue-to-pr
- Description: "Carry a GitHub issue to a verified PR with honest CI state."
- File Path: skills/github-issue-to-pr/SKILL.md

### github-issues
- Description: "Create, triage, label, assign GitHub issues via gh or REST."
- File Path: skills/github-issues/SKILL.md

### github-pr-workflow
- Description: "GitHub PR lifecycle: branch, commit, open, CI, merge."
- File Path: skills/github-pr-workflow/SKILL.md

### github-repo-management
- Description: "Clone/create/fork repos; manage remotes, releases."
- File Path: skills/github-repo-management/SKILL.md

### grounded-citations
- Description: "Ground answers and documents in cited, verifiable sources."
- File Path: skills/grounded-citations/SKILL.md

### huggingface-hub
- Description: "HuggingFace hf CLI: search/download/upload models, datasets."
- File Path: skills/huggingface-hub/SKILL.md

### improve-animations
- Description: Survey a codebase's animation and motion code as a senior motion advisor, then produce a prioritized audit and self-contained implementation plans for other agents (or cheaper models) to execute. Read-only on source code — it plans improvements, it does not apply them. Use when the user asks to "improve the animations", "audit the motion", "make this app feel better", or wants a roadmap of animation fixes rather than a review of a single diff.
- File Path: skills/improve-animations/SKILL.md

### improve-ui
- Description: Audit an existing product surface against its own design evidence, identify verified UI problems, and write self-contained implementation plans for another agent. Strictly read-only on product source. Use when asked to review, refine, improve, or clean up an interface without replacing its identity; investigate design-system drift; or prepare a design handoff.
- File Path: skills/improve-ui/SKILL.md

### jattana-agent-workflow
- Description: Use when coding in JATTANA GROUP with Codex, Claude Code, or Antigravity. Apply shared rules and verify changes.
- File Path: skills/jattana-agent-workflow/SKILL.md

### landing-page-conversion-audit
- Description: "Use when auditing landing pages."
- File Path: skills/landing-page-conversion-audit/SKILL.md

### marketing-plan
- Description: "Build comprehensive AARRR marketing plans at fractional-CMO level."
- File Path: skills/marketing-plan/SKILL.md

### meeting-action-items
- Description: "Turn meeting notes into cited decisions, owners, tickets."
- File Path: skills/meeting-action-items/SKILL.md

### multi-phase-prd-hardening
- Description: "Use when closing multiple PRD phases with one final commit."
- File Path: skills/multi-phase-prd-hardening/SKILL.md

### online-game-server-networking
- Description: "Use for authoritative multiplayer WebSocket servers."
- File Path: skills/online-game-server-networking/SKILL.md

### open-code-review
- Description: >
- File Path: skills/open-code-review/SKILL.md

### open-code-review-delegate
- Description: >
- File Path: skills/open-code-review-delegate/SKILL.md

### phase-based-simulation-development
- Description: "Use when implementing a spec-defined simulation/game phase."
- File Path: skills/phase-based-simulation-development/SKILL.md

### pick-ui-library
- Description: Pick the right library for a given frontend task from a curated, opinionated list — numbers, OTP inputs, charts, command menus, virtualization, drag and drop, toasts, state, styling, and more. Only runs when explicitly invoked; it does not trigger on its own.
- File Path: skills/pick-ui-library/SKILL.md

### plan
- Description: Write a markdown plan to .hermes/plans/; no execution.
- File Path: skills/plan/SKILL.md

### popular-web-designs
- Description: 54 real design systems (Stripe, Linear, Vercel) as HTML/CSS.
- File Path: skills/popular-web-designs/SKILL.md

### prd-acceptance-review
- Description: Use when auditing an implementation against PRD acceptance.
- File Path: skills/prd-acceptance-review/SKILL.md

### prd-phase-closure
- Description: "Use when finishing a PRD/TDD phase to acceptance criteria."
- File Path: skills/prd-phase-closure/SKILL.md

### product-marketing
- Description: "Create product marketing context: positioning, ICP, target audience."
- File Path: skills/product-marketing/SKILL.md

### production-hardening-review
- Description: "Use when hardening harnesses for production."
- File Path: skills/production-hardening-review/SKILL.md

### production-regression-audit
- Description: "Use after deploys to re-audit sibling paths and live health."
- File Path: skills/production-regression-audit/SKILL.md

### prototype
- Description: Build multiple genuinely different versions of a UI piece you describe, rendered behind a visual picker so you can flip through them live and promote the one that feels right. Only runs when explicitly invoked; it does not trigger on its own.
- File Path: skills/prototype/SKILL.md

### python-debugpy
- Description: "Debug Python: pdb REPL + debugpy remote (DAP)."
- File Path: skills/python-debugpy/SKILL.md

### railway-deploy-ops
- Description: "Railway deploy/verify: CLI, secrets, deploy status, health."
- File Path: skills/railway-deploy-ops/SKILL.md

### react-supabase-bug-audit
- Description: Use when auditing a React+Supabase app for runtime bugs.
- File Path: skills/react-supabase-bug-audit/SKILL.md

### review-animations
- Description: Reviews animation and motion code against a high craft bar derived from Emil Kowalski's design engineering philosophy. Default to flagging; approval is earned.
- File Path: skills/review-animations/SKILL.md

### seo-audit
- Description: "Audit SEO: technical, on-page, rankings, Core Web Vitals, indexing."
- File Path: skills/seo-audit/SKILL.md

### serial-bug-fix-closure
- Description: "Use for ordered multi-bug fixes with regression gates."
- File Path: skills/serial-bug-fix-closure/SKILL.md

### sketch
- Description: "Throwaway HTML mockups: 2-3 design variants to compare."
- File Path: skills/sketch/SKILL.md

### spike
- Description: "Throwaway experiments to validate an idea before build."
- File Path: skills/spike/SKILL.md

### supabase-app-engineering
- Description: "Supabase: offline queues, edge functions, vitest."
- File Path: skills/supabase-app-engineering/SKILL.md

### supabase-ops
- Description: Provision Supabase projects via CLI + Management API.
- File Path: skills/supabase-ops/SKILL.md

### system-design-knowledge-curation
- Description: "Use when curating shared system-design knowledge."
- File Path: skills/system-design-knowledge-curation/SKILL.md

### systematic-debugging
- Description: "4-phase root cause debugging: understand bugs before fixing."
- File Path: skills/systematic-debugging/SKILL.md

### test-driven-development
- Description: "TDD: enforce RED-GREEN-REFACTOR, tests before code."
- File Path: skills/test-driven-development/SKILL.md

### thai-app-localization
- Description: Use when building apps for Thai users.
- File Path: skills/thai-app-localization/SKILL.md

### timezone-aware-data-audit
- Description: "Use for timezone-aware backend date audits."
- File Path: skills/timezone-aware-data-audit/SKILL.md

### typescript-monorepo-foundation
- Description: "Use when bootstrapping a TypeScript pnpm monorepo."
- File Path: skills/typescript-monorepo-foundation/SKILL.md

### ui-skills-root
- Description: Use before UI-related work to select the smallest useful UI Skills context through the ui-skills CLI.
- File Path: skills/ui-skills-root/SKILL.md

### understand-chat
- Description: Use when you need to ask questions about a codebase or understand code using a knowledge graph
- File Path: skills/understand-chat/SKILL.md

### understand-diff
- Description: Use when you need to analyze git diffs or pull requests to understand what changed, affected components, and risks
- File Path: skills/understand-diff/SKILL.md

### understand-explain
- Description: Use when you need a deep-dive explanation of a specific file, function, or module in the codebase
- File Path: skills/understand-explain/SKILL.md

### understand-onboard
- Description: Use when you need to generate an onboarding guide for new team members joining a project
- File Path: skills/understand-onboard/SKILL.md

### web-app-smoke-testing
- Description: "Verify site logins fast; handles SPA click quirks."
- File Path: skills/web-app-smoke-testing/SKILL.md

### weekly-review-planning
- Description: "Weekly reset: commitments, stalled work, next-week plan."
- File Path: skills/weekly-review-planning/SKILL.md

### world-first-sandbox-game-development
- Description: Use for serial evidence-driven sandbox game development.
- File Path: skills/world-first-sandbox-game-development/SKILL.md

### xlsx
- Description: Create, read, edit Excel .xlsx workbooks and CSVs.
- File Path: skills/xlsx/SKILL.md

