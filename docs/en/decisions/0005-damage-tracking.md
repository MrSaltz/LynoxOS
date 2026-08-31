# 0005 — Compositor damage tracking: partial blit, not region expansion

> 🇧🇷 [Ler em Português](../../pt-br/decisions/0005-damage-tracking.md)

**Status**: Decided (Advanced Window System milestone, live desktop
session stabilization). **Area**: [window system](../architecture/window-system.md).

## Context

The compositor originally redrew every visible window on every single
frame, regardless of whether anything had actually changed — correct, but
wasteful once the system needed to sustain interactive frame rates.
Introducing damage tracking (recording which screen regions actually
changed since the last frame, and only redrawing those) was meant to fix
that, but the first implementation attempt introduced a real, user-visible
correctness bug: UI elements (a taskbar, status text) would intermittently
disappear depending on cursor position.

## Alternatives considered

- **(a) Per-window intersection test, redraw the whole window if it
  intersects the damaged region at all.** The initial approach. It broke
  z-order compositing: redrawing a full-screen background window without
  also redrawing whatever legitimately sits on top of it *in that same
  region* let the background silently paint over foreground elements
  whenever the two happened to share any damaged area — which, for a
  full-screen window, is essentially always (the cursor moving anywhere
  on screen counts as damage that a full-screen window "intersects").
- **(b) Expand the tracked damage region to cover a window's entire bounds
  once any part of it registers as touched, iterating bottom-to-top.**
  This fixed the disappearing-elements bug (correctness restored), but
  reintroduced the exact performance problem damage tracking was meant to
  solve: because the background window is full-screen, expanding damage to
  its full bounds effectively poisons the tracked region back to the whole
  screen on nearly every frame.
- **(c) For each window, compute only the exact overlapping sub-region
  between that window and the damaged area, and redraw *just that clipped
  sub-region* — no region expansion at all.** Bottom-to-top drawing order
  naturally repaints the correct final result within that clipped area,
  since anything sitting on top of the background gets its own turn to draw
  over the same pixels.

## Decision

**(c)** — clip each window's redraw to its exact intersection with the
damaged region, with no expansion step. Verified interactively: elements
stopped disappearing, and the frame rate recovered from the regression
introduced by (b).

## Consequences

- This is the version currently running: a real, measured frame rate in
  the ~15–20 FPS range on an emulated, unaccelerated framebuffer — a known,
  accepted limitation of software rendering at this stage, not a
  regression waiting to be fixed by revisiting this specific decision (see
  [desktop.md](../architecture/desktop.md#rendering-frame-pacing-and-a-live-fps-counter)).
- The general lesson generalized beyond this specific bug: whenever a fix
  is verified only for correctness, it's worth explicitly re-checking that
  the original performance property the change was meant to preserve
  actually still holds — (b) *looked* like a natural, incremental fix over
  (a), and it was already a regression from the performance goal before
  anyone measured it directly.
