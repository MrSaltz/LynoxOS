# UI Toolkit

> 🇧🇷 [Ler em Português](../../pt-br/architecture/ui.md)

Covers the UI Interaction phase (M33) and the in-house widget toolkit built
alongside the [window system](window-system.md): the layer applications
actually use to build their interface, rather than calling graphics
primitives (see [graphics.md](graphics.md)) directly.

## An in-house widget toolkit

Rather than every application drawing its own buttons and labels by hand,
Lynox has a small, reusable widget toolkit: a shared `Theme` (colors,
spacing) that every widget draws against, a base `Widget` behavior every
control implements, and a set of concrete widgets — `Label`, `Panel`
(a container that lays out heterogeneous widgets), `Button`, `TitleBar`,
and `Menu`. Every one of these draws through the same [`Surface`
abstraction](graphics.md#surfaces-one-abstraction-two-destinations) as
everything else in the graphics stack — a widget has no idea whether it's
being drawn into a window's live surface or an offscreen buffer for a test.

## Interaction model

A widget being drawn correctly and a widget *responding* to input correctly
are different problems, addressed as their own step:

- **Keyboard focus** — exactly one widget at a time can hold focus, and
  only the focused widget receives key events; visually, a focused widget
  is distinguishable from one that merely has the mouse over it.
- **Double-click** — recognized by a widget-level time-window-plus-distance
  check (a second click within a short time and a small pixel radius counts
  as a double-click), evaluated as a pure, independently testable function
  rather than something baked into the input-handling loop.
- **Pointer capture** — once a button is pressed, it stays in the "pressed"
  visual and logical state until the mouse button is released, even if the
  cursor briefly leaves the widget's area first — matching how real UI
  toolkits behave, and avoiding a class of bug where a drag that leaves a
  button's bounds is misread as a release.
- **Unified input handling** — mouse and keyboard input are funneled through
  one entry point per widget, rather than every widget implementing its own
  separate mouse-handling and keyboard-handling code paths.

## Where this shows up later

- Window chrome (title bars, menus, popups — see
  [window-system.md](window-system.md#window-chrome-and-lifecycle)) is
  built from these same widgets, not separate one-off drawing code.
- The desktop's built-in applications and the taskbar (see
  [desktop.md](desktop.md)) are the toolkit's real, load-bearing
  consumers — every one of them is built from `Panel`/`Label`/`Button`
  rather than hand-rolled drawing code.
- See [`docs/en/design/ui-guidelines.md`](../design/ui-guidelines.md) for
  the visual and interaction principles the toolkit follows.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M33).
