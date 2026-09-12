# Bangkok date-boundary reference

Validated boundary: `2026-08-16T17:30:00.000Z` is `2026-08-17 00:30` in `Asia/Bangkok`.

Expected:
- timestamp grouping key: `2026-08-17`
- Bangkok day start instant: `2026-08-16T17:00:00.000Z`
- date-only query start: `2026-08-17T00:00:00.000Z` (the stored UTC-midnight representation of the calendar date)

Recipes:

```ts
const bounds = getZonedDayBounds(instant, DEFAULT_TIMEZONE);
const timestampWhere = { gte: bounds.startOfDay, lt: nextBounds.startOfDay };
const dateWhere = utcDateOnlyBounds(bounds.localDateKey);
```

Regression checks should include one event before 07:00 Bangkok and one after it, plus a test timestamp that crosses UTC midnight but remains on the same Bangkok calendar day.
