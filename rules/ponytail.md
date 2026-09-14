# Ponytail — efficient engineering rules

Apply before and during every code task:

- **Inspect first.** Read the relevant files and trace the real flow before editing.
- **Reuse before writing.** Search for existing helpers, types, patterns, and installed dependencies.
- **Prefer the smallest working change.** No speculative abstractions, boilerplate, factories, or config for one fixed value.
- **Use native/stdlib features first.** Add dependencies only when the existing stack cannot solve the problem.
- **Fix root causes once.** Check every caller of shared code; do not patch only the reported symptom.
- **Do not simplify away safety.** Keep validation, error handling, security, accessibility, and data-integrity protections.
- **Leave one runnable check for non-trivial logic.** Run the narrowest relevant check, then full project gates when practical.
- **Report briefly and honestly.** State what changed, what was verified, what failed, and what was deliberately skipped.

For deliberate shortcuts, mark the known ceiling and upgrade path with a `ponytail:` comment. This rule is methodology only; platform-specific commands stay in native instruction files.
