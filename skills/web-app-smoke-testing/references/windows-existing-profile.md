# Windows existing-profile browser attachment

When Hermes Computer Use must control an already signed-in Chrome profile, Chrome, `cua-driver.exe`, and the Hermes gateway must run in the same interactive Windows session.

## Root cause pattern

A Hermes gateway registered as a Scheduled Task with a `BootTrigger` and `LogonType=S4U` runs in Session 0. Chrome normally runs in the logged-in user's Session 1. Cua rejects the mismatch with an interactive-session error.

## Fix

Use a Scheduled Task `LogonTrigger` with `LogonType=InteractiveToken`, then stop the old gateway and start the task after the user is logged in. Keep `computer_use.grant_existing_profile=true` for attaching the existing Chrome profile.

## Verify

- `quser`: identify the active interactive session.
- `Get-CimInstance Win32_Process`: check `SessionId` for gateway, `cua-driver.exe`, and Chrome; they must match.
- Run a fresh Computer Use capture only after the process sessions match.
- If a Chrome key combo is rejected in background mode, follow the tool escalation and retry the same action in foreground mode, then capture again.

A non-login URL is not proof of authentication; verify rendered user-specific UI/data.