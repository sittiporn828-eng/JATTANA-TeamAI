# Final-closure techniques

## Alert acceptance probe

A dashboard returning alert names is not enough. Keep an injected `alertSink`, production webhook/dispatcher wiring, and an executable forced-condition probe. For a server-down probe, run the API with `GAME_SERVER_HTTP_URL` pointed at an unavailable endpoint, call `/metrics/dashboard`, and assert `alerts` contains `server_down`. The same dashboard should evaluate `low_tick_rate`, `high_disconnects`, `matchmaking_failure`, and `payment_failure` from live counters/probes.

## Determinism input replay

A contract containing `inputs` is not evidence if the test only steps the world. Export/use the actual competitive input application path before stepping at each recorded tick, then hash canonical checkpoints including the applied input/state. Replay the same contract twice and assert identical hashes. A minimal state-log path is acceptable only when it is the production simulation input seam, validates tick ordering, and is covered by a regression test.

## Localization source coverage

Locale switching is incomplete while rendered JSX reads a fixed-locale dictionary, raw power IDs, or literals such as quality presets and notification controls. Use the selected locale for app chrome, key every visible status/control, and map data-driven identifiers (powers, modes, statuses) through locale keys with an explicit fallback policy. Build/typecheck validates shape; also scan rendered source and review both locales for every visible key.

## Commit/push closure

After the fresh review returns `passed:true` and the final gates pass, inspect `git status --short --untracked-files=all` for generated rehearsal artifacts before staging. Commit the complete verified diff, push the intended branch, then verify all three: clean working tree, local `HEAD`, and `git ls-remote origin refs/heads/<branch>` are identical. If SSH push fails, inspect `~/.ssh/config` and test configured GitHub host aliases with `ssh -T`; use the authenticated alias that matches the repository owner rather than disabling host verification or exposing keys.
