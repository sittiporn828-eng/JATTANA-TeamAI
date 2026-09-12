# Godforge full-flow closure example

## Defects exposed only by live seams

- Allocation accepted room duration and power bans but the authoritative game server ignored them. Validate at allocation and pass the validated values into match construction; test behavior through the HTTP allocation route.
- The game server emitted `winnerPlayerId`/`victoryType` while the API expected snake_case. Injected outcome stubs hid the mismatch. A real local HTTP producer regression exposed it.
- Outcome resolution must return `running` to the route so it can answer `409 MATCH_NOT_FINISHED`; transport/config failures remain `503`. Bind the outcome’s `match_id` to the expected match.
- Rehearsals that posted client-declared winners became invalid after authoritative settlement. Replace them with real WS ready/action, assert premature settlement rejection, then clean up via remake/release.
- Load smoke should not duplicate the full ranked lifecycle. Use a small authenticated status preflight, while the dedicated E2E owns lifecycle coverage.

## Browser findings

- Localized `<option>` labels used as implicit values changed Quality from Medium to Low after switching language. Give options stable enum values and localize only labels.
- Global `p/l` shortcuts hijacked Ctrl/Meta/Alt browser shortcuts and editable controls. Ignore modifiers plus input/select/textarea/contentEditable targets centrally.
- GM Console was initially omitted from “every flow.” Its dev server lacked an API proxy, and login immediately fetched dashboard with stale React session state. Test every runnable UI; configure a native Vite proxy and use the freshly returned login token for the first dashboard request.

## Verified flow set

- Player UI: Home, Lobby, Play/Sandbox, Loadout, Settings; pause/resume/speed/power/language/accessibility/shortcuts/console.
- Backend: guest, queue, allocation, two WS ready, authoritative power broadcast, unfinished result rejection, remake, release.
- Operations: migrations, ranked maps, readiness/metrics, load smoke, kill switch, signed local webhook.
- Gap routes: tutorial, store, season-pass claim, missing receipt, admin dashboard/moderation validation, replay input, logout revocation.
- GM UI: login, dashboard, matches, players, rooms, queue, operations, audit, feature flags, logout.

Never record real tokens or session values in evidence.