# Lessons from the meekamrai QA closure session (24 Aug 2026)

## SQL-function fixes (stored procedures)

- **Patch, don't rewrite.** Extract the original function body from its
  migration file and string-replace only the offending block. A full rewrite of
  `admin_receive_purchase_order` silently dropped audit-log inserts, supplier
  price history, and the richer received/partially_received status transition —
  it was scrapped mid-session and replaced with an extracted-body patch.
- Verify signatures match the original exactly: parameter names, DEFAULT values,
  return shape. Callers break on renamed params even with identical types.
- Syntax-check multi-migration batches safely against prod by wrapping in
  `BEGIN; ... ROLLBACK;` via the Management API query endpoint before applying.
  Empty response `[]` = batch parses and executes; nothing persists.

## Client/server unit-semantics drift

- When both sides compute the same money total (flat vs multiplied addons,
  batch vs per-serving prices), write one small node script implementing BOTH
  formulas and assert equality over representative cases BEFORE touching SQL —
  proves the fix matches without seeded prod data.
- After refactoring shared derived values across React `useMemo`s: tsc does NOT
  catch a temporal-dead-zone reference from an earlier memo to a symbol defined
  in a later memo — it crashes at render only. Diagnostic shortcut: if a test
  fails with the change applied but passes with the change stashed, bisect
  newly-introduced symbol references first (this found a TDZ crash that a full
  vitest run surfaced as one opaque missing-text failure).

## Deploy verification on Cloudflare Pages

- Auto-deploy fires on git push, but the new asset hash can take several
  minutes to appear on pages.dev. Grepping minified bundles for source-level
  function names is unreliable (minifier renames symbols) — prefer checking
  whether the served index.html hash changed vs. the pre-push hash, and treat
  "hash changed + local build passed" as sufficient evidence unless a runtime
  smoke test is available.
