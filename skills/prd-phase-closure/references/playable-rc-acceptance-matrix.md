# Playable RC Acceptance Matrix — Reusable Template

Use this when a large game PRD needs a bounded playable stopping point.

## P0 row shape

| ID | Acceptance | Authoritative producer/state | Runtime consumer | Focused test | Browser/manual evidence | DoD | Deferred |
|---|---|---|---|---|---|---|---|
| P0-xx | One user-visible behavior | Simulation/server field or event | Renderer/UI path | Exact runnable check | Smallest real flow + console | Gate + clean push | Explicit non-goal |

## Required P0 baseline

- Home → Sandbox
- deterministic simulation tick
- camera zoom/pan
- power select → target → cast
- HUD/state synchronization
- readable entities, settlements, resources, and consequences
- Low/Medium/High presentation presets
- runtime accessibility settings
- runtime locale switch
- browser console health
- full local gates
- clean pushed tree

## P1 transport seam checklist

- Browser config contains canonical WebSocket URL, match ID, player ID, and canonical civilization ID.
- `player_ready` uses the server's exact civilization identifier; reject invalid client configuration before sending.
- `world_snapshot` and `entity_delta` are parsed as untrusted data, not TypeScript-cast blindly.
- A complete world projection is validated before applying it; malformed/partial payloads do not change state.
- Accepted snapshot and delta ticks are non-negative integers and strictly monotonic across both message types.
- Local simulation stepping stops once authoritative transport is configured; reconnect does not re-enable local stepping.
- Session token is retained for reconnect; reconnect uses increasing client sequence numbers.
- Server integration tests and browser-client runtime evidence are separate claims.
- Missing live backend/browser evidence is reported as an evidence gap, not promoted from build/test output.

## Scope freeze

Once P0 rows pass and the tree is clean/pushed, freeze the playable scope. Put production hardening in P1; do not reopen visual implementation for optional polish or evidence stronger than the PRD requires.
