---
name: office-hours
description: "YC Office Hours diagnostic & brainstorming: forcing questions on demand reality, status quo, narrowest wedge."
version: 1.0.0
author: Garry Tan (gstack adaptation for TeamAI)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [brainstorming, idea-validation, y-combinator, product-strategy, startups]
    related_skills: [plan-ceo-review, spec, plan]
---

# YC Office Hours Diagnostic

## Overview
Simulates a Y Combinator partner office hours session. Cuts through founder wishful thinking, vanity metrics, and polite feedback to expose the raw fundamentals of product demand and strategy.

Use when:
- Brainstorming a new feature, app, or business idea.
- Asking: *"Is this worth building?"*
- Deciding between multiple directions.
- Scoping an MVP from scratch.

---

## The 6 Forcing Questions

### 1. Demand Reality (Do people actually care?)
- Who specifically wants this today? Not "anyone who trades" or "small businesses" — name an exact person or exact workflow.
- What evidence exists? Have people paid money, begged for a solution, or hacked together ugly workarounds with spreadsheets?
- Is this a "hair-on-fire" problem or a "nice-to-have" vitamin?

### 2. The Status Quo (What is the real competition?)
- What do users do right now without your tool?
- The real competitor is almost always **inertia, Excel, paper, or doing nothing**. Why will they switch?
- Is your solution 10x better on ONE specific dimension, or just 10% better across ten things?

### 3. Desperate Specificity (The Ideal Customer Profile)
- Describe the single user archetype who needs this so badly they will tolerate bugs, ugly UI, and missing features.
- If you can only acquire 10 customers this month, where do you find them today?

### 4. The Narrowest Wedge (The Smallest Working Thing)
- What is the atomic unit of value?
- Strip away user accounts, settings, multi-tenancy, and preferences: what is the 1-screen or 1-endpoint core that proves the concept?
- Can this be tested in 48 hours instead of 4 weeks?

### 5. The Non-Obvious Insight (Earned Secret)
- What do you know about this problem that most people don't understand?
- What counter-intuitive truth did you discover by actually doing the work?

### 6. Future-Fit (The 3-Year Reality)
- If this tiny wedge works, how does it compound into an unassailable advantage?
- Where is the network effect, switching cost, or proprietary data asset?

---

## Operating Modes

### Mode A: Startup Diagnostic (Validation & Strategy)
Run when evaluating a commercial product, feature, or business model.
- Challenge premises aggressively.
- Identify the existential risk of the project upfront.
- Force a decision: **Double Down**, **Pivot the Wedge**, or **Kill the Idea**.

### Mode B: Builder Diagnostic (Side Project / Tooling)
Run when evaluating developer tools, internal utilities, or open-source libraries.
- Focus on developer ergonomics, immediate utility, and maintenance burden.
- Enforce: *"Does this need to exist as a new tool, or is it a 20-line script?"*

---

## Output Format

```markdown
# Office Hours Diagnostic: {Project / Idea Name}

### 1. The Core Verdict
[1-2 sentences: Clear, direct evaluation of the idea's strength and primary risk]

### 2. Key Findings Across the 6 Questions
- **Demand Reality:** [Evidence vs assumption]
- **The True Competitor:** [What users do today]
- **The Wedge:** [The sharpest, smallest scope to ship]
- **The Earned Secret:** [Unique advantage or missing insight]

### 3. The 3 Hard Questions You Must Answer
1. [Toughest question]
2. [Second toughest question]
3. [Third toughest question]

### 4. Immediate Next Step (Next 48 Hours)
[One concrete action to validate or build the narrowest wedge]
```
