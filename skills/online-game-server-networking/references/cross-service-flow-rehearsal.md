# Cross-service authoritative flow rehearsal

Use this when unit tests inject allocator/outcome stubs but the real API → game-server seam has changed.

## Minimal live rehearsal

1. Build producer and consumer workspaces before starting either process.
2. Start a fresh isolated game-server and API with matching allocate/release/result tokens, unique ports, and a unique temporary persistence path.
3. Probe readiness.
4. Exercise the real sequence:
   - create two identities
   - queue and match
   - verify allocation in game-server metrics
   - open two WebSockets
   - send `player_ready` for both authorized player IDs
   - observe snapshots
   - send one `use_power`
   - verify the peer receives an authoritative power event owned by the bound player
   - submit settlement before finish and require the documented unfinished response
   - call remake/release and verify allocation returns to available
5. Always close sockets in `finally`; otherwise a failed assertion can leave the script hanging.
6. If a rehearsal fails after manual control-plane intervention, restart both API and game-server before rerunning. Releasing only the game-server can leave API in-memory match/settlement state inconsistent with the server.

## Contract-drift test pattern

Injected `resolveOutcome` stubs can agree with the API while disagreeing with the real HTTP producer. Add one integration test using a tiny native `node:http` server that returns the exact producer JSON. Assert the public API status/code, not parser internals.

For result settlement:

- Producer and consumer must use one exact field convention.
- Include `match_id` in the result response and require it to equal the expected allocation before accepting outcome data.
- A reachable authoritative match with `status: running` is an unfinished-match response, not infrastructure unavailability.
- Network/auth/non-2xx/malformed/mismatched-match failures remain unavailable/fail-closed.

## Allocation settings

A validated allocation field is not implemented merely because the HTTP route accepts it. Trace each setting into match construction and leave a behavioral regression:

- short duration must alter `durationTicks` using the simulation TPS, including any defined sudden-death period;
- power bans must be validated before allocation state mutates and passed into authoritative loadout/match validation;
- reject unknown, non-bannable, duplicate, or fixed-loadout-conflicting bans before entering `running`.

## Script dependencies

For Node 22+ rehearsal scripts, prefer the platform `WebSocket` rather than adding a root dependency solely for a script. Use `addEventListener`/`removeEventListener` and read message payload from `event.data`.