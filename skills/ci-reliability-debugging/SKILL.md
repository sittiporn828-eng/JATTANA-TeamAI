---
name: ci-reliability-debugging
description: "Use when CI exposes flaky async tests; fix races remotely."
version: 1.0.0
license: MIT
metadata:
  hermes:
    tags: [ci, flaky-tests, async, websocket, integration-testing, github-actions]
    related_skills: [systematic-debugging, test-driven-development, github-pr-workflow]
---

# CI Reliability Debugging

Use when local tests pass but GitHub Actions or another CI runner fails intermittently, especially for WebSocket, timers, queues, or event-stream assertions.

## Workflow

1. **Check the remote run first.** Identify the exact commit, workflow run, failing job, test, assertion, and received/expected event sequence. Local success is not evidence that CI passed.
2. **Read producer and test together.** Trace the event path from the server timer/queue/socket handler to the test observation and locate the synchronization boundary.
3. **Build a tight local repro.** Run the narrow test and repeat it when timing-sensitive. Keep the original assertion; do not weaken it to make it green.
4. **Fix synchronization at the observation seam.** If the test needs an async or periodic event, wait for that named event with a bounded polling helper or promise. Assert only after observing it.
5. **Avoid fake fixes.** Do not add arbitrary sleeps, inflate timeouts without understanding the producer, reorder unrelated production code, or remove the expected event.
6. **Run local gates:** `pnpm format:check`, `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, and `git diff --check`.
7. **Push and verify remote CI.** Query the run for the exact pushed commit and poll until `completed`; report the actual result. A reliable CLI pattern is `gh run list --repo OWNER/REPO --commit SHA --limit 1`, repeated until the row begins with `completed`. Never claim CI passed from local gates alone.

## Reporting rule

Separate three claims: local gates, remote CI, and repository sync. Say `local passed` until the GitHub run for the pushed commit is completed successfully; only then say `CI passed`. Verify `HEAD == origin/main` and a clean working tree independently.

## Async/WebSocket rule

A server may emit events on independent schedules. A `world_snapshot` sent synchronously after `player_ready` does not imply that the next timer-driven `entity_delta` has arrived. Use an event-aware helper such as `waitFor(messages, 'entity_delta')`, then inspect message types. This is more reliable than `setTimeout(50)` and documents the protocol contract.

## Verification checklist

- [ ] Remote failure explained from producer/test timing
- [ ] Exact failing test passes repeatedly locally
- [ ] Fix waits for a semantic event, not elapsed time
- [ ] Full local quality gates pass
- [ ] New commit pushed
- [ ] GitHub Actions run for that commit is completed and successful
- [ ] `HEAD == origin/main` and working tree is clean

See `references/websocket-ci-race.md` for the validated event-wait pattern and evidence shape.
