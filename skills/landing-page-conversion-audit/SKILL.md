---
name: landing-page-conversion-audit
description: "Use when auditing landing pages."
---

# Landing Page Conversion Audit

Use this skill for a deployed landing-page review when the user wants a practical check against a product/UX plan. Produce evidence-backed findings, not a generic design opinion.

## Scope

Audit one public landing page and compare the rendered/user-visible experience with the supplied brief or current product promise. Keep the work read-only unless the user separately asks for implementation.

## Workflow

1. **Load the brief first.** Extract the intended positioning, section order, CTA vocabulary, trust claims, calculator behavior, screenshot policy, and Definition of Done. Treat proposed alternatives as options, not simultaneous requirements.
2. **Inspect the deployed page.** Use browser rendering/snapshot plus a clean text extraction fallback. Record title, hero copy, CTA labels/targets, section order, visible disclaimers, screenshots/mockups, FAQ, and final CTA.
3. **Test the conversion spine.** Check Hero → calculator, calculator → signup, primary CTA → signup, secondary CTA → calculator, navigation anchors, and FAQ interaction when the environment permits. Check console/runtime errors after meaningful interactions.
4. **Check consistency across the page.** Search every CTA label and compare it with the plan's primary/secondary system. Compare repeated examples and prices across sections. Check whether product capabilities are qualified consistently, especially third-party integrations.
5. **Separate real screenshots from mock UI.** If screenshots are placeholders, verify that the page labels them as examples/mockups and does not imply real customer data or production telemetry. Never recommend fabricated social proof.
6. **Review responsive risk.** Inspect desktop and narrow mobile rendering when browser control is available. Look for horizontal overflow, clipped CTAs, unreadable tables/screenshots, excessive Hero height, and touch targets. If rendering cannot be verified, report it as unverified rather than a finding.
7. **Rank findings.** Prioritize conversion blockers and trust/confusion issues, then content consistency, then polish. Stop at the smallest useful set, normally no more than five. Each finding must include URL/section, evidence, expected behavior, actual behavior, impact, and one deterministic correction.
8. **Report honestly.** State what was tested, what was not, and any access/tool limitation. Do not convert a failed probe or bot-blocked HTTP request into a product bug when browser-rendered evidence says otherwise.

## Default decision rules

- Keep a strong existing headline if it communicates the same value proposition; do not force a preferred wording merely because it was listed as Option A.
- A CTA system is inconsistent when equivalent actions use multiple labels without a deliberate distinction. Recommend one primary signup label and one calculator label.
- A five-step explanatory loop may be valid, but compare it to the brand's canonical four-step loop and explicitly call out the hierarchy decision instead of silently treating it as a bug.
- Example numbers must either be clearly separated as different scenarios or be made internally consistent across sections.
- Placeholder screenshots are acceptable during pre-production only when clearly marked as mock/demo/example and never presented as real customer evidence.
- Absence of social proof is not a defect when the plan says not to invent numbers; recommend adding it only when verified evidence exists.

## Output

Use this compact structure:

```markdown
## Verdict
<overall readiness and the strongest evidence>

## Findings
### P0/P1 — <title>
- Location:
- Evidence:
- Impact:
- Fix:

## Verified strengths
- ...

## Not verified / intentionally deferred
- ...
```

Do not modify product source during an audit. If the user asks to implement a selected finding, hand off to the appropriate coding skill and preserve the audit evidence.

See `references/conversion-audit-checklist.md` for the reusable checklist and evidence fields.
