# Phase 10 live-seam notes

Use these checks when reviewing replay/post-match work:

- A replay `capture()` call reachable only from an admin route is not production wiring. Capture when the ranked/custom allocation succeeds, retain the authoritative match ID and seeds, then append `match_finished` from the authoritative terminal result path.
- `resimulate()` must execute a deterministic reducer/simulation over the stored seed, initial snapshot, and canonical input stream. Returning the same Map arrays twice only proves storage stability. A reducer that appends or returns the stored event array is still an event echo. Filter terminal events deliberately and add a regression where changing/removing stored event data cannot change the seed+inputs result.
- Normalize the actual input schema used by the producer (`type` versus `kind`) before reducing. Tests must exercise the production-shaped input, not only a convenient reducer-shaped fixture.
- Post-match full-field round trips test adapter coverage, not truth. The terminal match/result path must write server-derived winner, score, rating, XP, and timeline fields; admin ingestion should be correction/import only.
- Ranked replay privacy must be enforced on every data endpoint (replay GET, seek, spectator join, and any feed), not only in `spectator.join()` metadata. Resolve ranked status from the authoritative live-match/replay registry using `match_id`; ignore or reject a client-supplied `ranked` flag. A client request for `ranked:false` must not remove the ranked delay or redaction.
- A per-actor streamer toggle is not acceptance if `view()` has no production caller. Apply `spectator.view(actor, frame)` at the actual response/transport boundary and test that the returned frame is redacted after enabling streamer mode.
- Unknown replay/spectator resources should produce the route's consistent 404 contract, not an uncaught service exception/500. Add an HTTP regression for the unknown ID path.
- Keep persistence and live spectator transport separate from the Phase 10 API contract when the PRD explicitly defers them; report them as deferred rather than silently treating an in-memory Map or metadata-only join policy as durable/live transport.
