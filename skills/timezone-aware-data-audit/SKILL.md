---
name: timezone-aware-data-audit
description: "Use for timezone-aware backend date audits."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [timezone, date-boundaries, Prisma, backend, regression-testing]
---

# Timezone-Aware Data Audit

Use for health, medication, analytics, calendar, caregiver, and notification flows where a user's calendar timezone differs from the server timezone.

## Core rule

Do not apply one date strategy to every column:

- **Timestamp columns** (`DateTime` with a real instant): query with zoned day bounds and group/format with the product timezone.
- **Date-only columns** (`@db.Date`, commonly represented as UTC midnight): keep the calendar key in the product timezone, but query using UTC-midnight bounds derived from that key.

For Thailand products, reuse the existing timezone helpers (`DEFAULT_TIMEZONE`, `getZonedDayBounds`, `formatInUserTimezone`) and `utcDateOnlyBounds`; do not create another date utility.

## Workflow

1. **Map the flow.** Trace caller → service → Prisma schema column → grouping/formatting → response/UI. Inspect every sibling caller before editing a shared helper.
2. **Classify columns.** Verify the Prisma schema/migration type for every date field. Mark each as timestamp or date-only.
3. **Make a boundary fixture.** Use an instant around local midnight, e.g. `2026-08-16T17:30:00Z` = `2026-08-17 00:30` in Bangkok. Assert the expected local key and query bounds.
4. **Patch the root.** For timestamps use `getZonedDayBounds`; for date-only values use `utcDateOnlyBounds(localDateKey)`. Keep grouping and query boundaries consistent with the column type.
5. **Scan siblings.** Grep touched files for `toISOString().split('T')[0]` and `toISOString().slice(0, 10)`. Keep ISO serialization for API instants, but replace it when it is used as a local calendar key.
6. **Test in layers.** Run focused boundary/regression tests first, then the full type-check, test suite, lint gate, and production build. Restore generated HTML/assets before commit.
7. **Verify deployment.** Confirm the pushed SHA, deployment status, and a live health endpoint after startup settles. Report unrelated queue degradation separately from the date fix.

## Common failure modes

- Using Bangkok instant bounds on a Prisma `@db.Date` field drops the current local date because the stored value is UTC midnight.
- Fixing one grouping loop while leaving weekly summaries, badge progress, caregiver trends, or notification dedupe on UTC keys leaves sibling paths broken.
- Updating tests with timestamps that cross Bangkok midnight can make a correct local-day implementation look wrong; choose fixtures inside the intended local day and add one explicit midnight-crossing assertion.
- A green test run before the final patch is not evidence; rerun all gates after the last edit.

## Reference

See `references/bangkok-date-boundary.md` for the validated boundary example and query recipes.
