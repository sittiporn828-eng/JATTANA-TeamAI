# Bottom dock browser evidence

## Pattern

For a world-first sandbox game, use a bottom dock with three layers: category tabs, active-category power tray, and compact navigation actions. This borrows a genre interaction pattern without copying WorldBox assets, branding, exact palette, or layout.

## Godforge implementation

- Categories: `World`, `Nature`, `Life`.
- Existing powers remain authoritative; the dock only changes `powerTab` and `selectedPower` presentation state.
- Existing `castPowerAt()` remains the only gameplay mutation path.
- Duplicate right-side power controls are hidden after the dock is introduced.
- Desktop `.play-area` is viewport-locked (`position: fixed; inset: 0; overflow: hidden`) so the dock stays anchored.
- Mobile overrides restore normal flow and horizontal/stacked controls for touch scrolling.

## Verification recipe

1. Start the explicit workspace dev stack.
2. Open the actual Vite URL and enter Sandbox.
3. Confirm category buttons, power buttons, selected state, and navigation actions in the accessibility snapshot.
4. Capture the Play route and inspect canvas/HUD/dock together.
5. In the browser console evaluate:

```js
({
  scrollHeight: document.documentElement.scrollHeight,
  viewportHeight: window.innerHeight,
  dock: document.querySelector('.play-dock')?.getBoundingClientRect(),
})
```

Desktop acceptance: `scrollHeight === viewportHeight`; dock bottom remains inside the viewport; no dock/HUD overlap; console has no runtime errors.

## Failure found and fixed

Initial dock CSS left the canvas at its intrinsic `min-height`, producing a page taller than the viewport and placing the dock below the visible screen. A successful build did not catch it. Viewport-locking the desktop Play surface fixed the runtime layout; mobile remains normal-flow.
