# Allocation contract and rehearsal hardening

Validated patterns from a whole-system authoritative game-server closure.

## Strict allocation boundary

Treat allocation as a typed wire contract, not a convenience parser:

- Require `room` to be a non-null, non-array object. Do not replace malformed input with `{}`.
- Require `match_duration_seconds` to already be a JSON number and integer. Do not use `Number(...)`; it accepts strings, booleans, and arrays.
- Bound duration at the HTTP boundary and again inside the authoritative server. Use the product's documented maximum; an unbounded integer can overflow `durationTicks` to `Infinity` and create a match that never ends.
- Require `power_bans` to be an array. Do not silently replace malformed values with `[]`.
- Validate every banned power against the authoritative catalog, ban eligibility, uniqueness, and required/default loadout constraints.
- Allocation participant IDs and room settings are authoritative inputs; propagate them into the simulation instead of validating and discarding them.

Minimal regression matrix: malformed `room`, numeric string duration, boolean duration, non-array bans, duration above maximum, and one valid custom duration/ban allocation.

## Cross-service result contract

A consumer test stub can repeat the consumer's wrong schema. Exercise the literal HTTP response shape from the game server, including field casing, and bind the response's `match_id` to the match being settled. On unavailable, unfinished, or mismatched outcomes, restore any acquired settlement lock before returning.

## Rehearsal timeouts

Every wait needs a bound, including WebSocket opening. A server may accept TCP while never producing `open` or `error`; without a handshake timeout, `Promise.all` hangs before reaching cleanup.

A safe open helper:

1. Install `open` and `error` listeners.
2. Start a short timeout.
3. On every completion path, clear the timer and remove both listeners.
4. Reject timeout so the outer `finally` closes all sockets.

Keep lifecycle E2E and load smoke separate: lifecycle owns allocation/join/action/result/release; load smoke uses a small authenticated preflight and repeated read/readiness traffic rather than inventing client-authoritative settlement.