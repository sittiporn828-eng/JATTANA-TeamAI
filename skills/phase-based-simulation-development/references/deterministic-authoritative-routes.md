# Deterministic authoritative route-state checklist

Use for vehicles, caravans, patrols, couriers, or any entity that advances through an indexed path.

## Domain contract

- Keep route ownership in authoritative simulation state; rendering only projects it.
- Store a cursor such as `routeOffset` when a route can repeat an index. Deriving progress with `indexOf(currentNode)` is ambiguous on out-and-back routes.
- Generate cyclic routes whose **every transition, including last → first**, is contiguous. For orthogonal tile roads, require Manhattan distance exactly `1`.
- Avoid a one-way route with modulo wrap unless its final node is adjacent to its first; otherwise the entity teleports at the seam. A minimal safe local loop is `outbound + reverse(outbound without endpoints)`.
- Bind route scope to the owner/economy boundary. A faction wagon should not silently traverse an enemy capital merely because roads share a row.
- If cargo is exposed as authoritative state, include an exact resource kind and derive its initial amount from owner stockpile. Loading/delivery mutation may remain a declared later economy ceiling, but do not present a free-floating constant as integrated logistics.

## Trust-boundary validation

Validate both member schema and cross-field invariants:

1. Exact entity kind/faction/resource-kind allowlists.
2. Finite non-negative cargo.
3. Integer, in-bounds route indices.
4. Non-empty route; current index belongs to it.
5. Cursor is in bounds and `route[cursor] === currentIndex`.
6. Current position exactly equals the referenced road position.
7. Every consecutive route pair, including wraparound, is contiguous.
8. Road members themselves have valid integer positions and exact bridge semantics.

## TDD and verification

- RED: generated route moves deterministically and remains road-bound.
- RED: malformed snapshot `[currentRoad, distantRoad]` is rejected even when cursor/index/position otherwise match.
- Probe multiple seeds and faction counts; report maximum route-step distance and replay equality.
- Rebuild the producer package before consumer tests in pnpm workspaces to avoid stale `dist` false greens.
- Browser-check High/Medium/Low projection, reduced-motion interpolation behavior, console, and frame timing.
- Any route/cursor/schema edit invalidates the prior review; rerun full gates and request a fresh fail-closed review.
