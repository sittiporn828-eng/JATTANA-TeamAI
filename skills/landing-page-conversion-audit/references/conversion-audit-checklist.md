# Conversion Audit Checklist

Use this as a compact evidence sheet for one deployed landing page.

## Brief extraction

- Core promise:
- Primary CTA and target:
- Secondary CTA and target:
- Canonical section order:
- Calculator inputs/outputs:
- Trust/screenshot policy:
- Explicit non-goals:

## Runtime evidence

- URL and timestamp:
- Desktop viewport checked:
- Mobile viewport checked:
- Page title:
- Hero headline/supporting copy:
- CTA labels and hrefs:
- Calculator interaction result:
- Signup path:
- Anchor/navigation results:
- FAQ interaction:
- Console errors:
- Screenshot/evidence paths:

## Consistency checks

- [ ] Primary signup CTA uses one label everywhere
- [ ] Calculator CTA uses one label everywhere
- [ ] Repeated prices/examples are the same or clearly scoped
- [ ] Third-party integrations are described with accurate limits
- [ ] Mock screenshots are visibly labeled as mock/demo/example
- [ ] Social proof is real or intentionally omitted
- [ ] Disclaimer distinguishes estimates from guaranteed results
- [ ] Mobile has no horizontal overflow
- [ ] Table, calculator, and CTA remain legible at narrow widths

## Finding format

```markdown
### P1 — Short title
- Location: section or URL
- Evidence: exact visible text/state and how it was observed
- Expected: brief contract from the plan or direct contradiction
- Actual: what the user sees
- Impact: conversion, trust, or comprehension consequence
- Fix: one deterministic correction
- Confidence: high/medium/low
```

## Evidence discipline

- Text extraction can confirm copy and ordering, but not pixel-level layout.
- A failed non-browser HTTP probe may be bot protection; do not call it a broken asset without rendered evidence.
- If browser interaction is unavailable, label responsive and functional checks as unverified.
- Do not report absence of social proof as a defect when the plan forbids invented numbers.
