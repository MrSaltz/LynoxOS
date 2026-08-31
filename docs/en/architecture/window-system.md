# Window System

> 🇧🇷 [Ler em Português](../../pt-br/architecture/window-system.md)

Covers the windowing and compositing part of the Graphics/Desktop phase
(M28–M32) and the Advanced Window System phase (M34): everything that turns
a collection of drawing surfaces (see [graphics.md](graphics.md)) into
independently movable, resizable, closable windows on screen.

## Windows as owned surfaces

Each window owns its own off-screen drawing surface (see
[graphics.md](graphics.md#surfaces-one-abstraction-two-destinations)) — an
application draws into its own window's surface without any awareness of
where that window sits on screen, whether it's currently visible, or what
other windows exist. That separation (an application only ever draws into
its own buffer) is what makes the compositor's job well-defined: it's
purely responsible for arranging already-rendered window contents into a
final frame, never for anything an application draws.

## The compositor and z-order

The compositor is responsible for taking every currently-visible window
(each with its own position, size, and drawing surface) and producing the
single final frame that gets presented to the real framebuffer. Windows are
maintained in a strict z-order (back to front) that also drives which
window a click lands on (a window on top wins), and special layers exist on
top of the normal z-order: modal windows block interaction with anything
behind them until dismissed, and always-on-top windows stay above ordinary
windows even after being unfocused, with modals still taking precedence
over always-on-top when both are present.

## Damage tracking

Recomposing every visible window into the final frame on every single
frame, regardless of whether anything actually changed, wastes work a
static desktop doesn't need to pay for. Damage tracking records which
*regions* of the screen were actually touched since the last frame, and the
compositor only redraws windows within those regions — and only the
specific overlapping *portion* of each window, not the whole window,
which matters most for a full-screen window like the desktop background:
naively redrawing "the whole window" every time it merely *overlaps* a
tiny bit of damage (which a full-screen window always does) would silently
erase the performance benefit damage tracking was built to provide. Getting
this right took two iterations — the first fix (expanding the damage region
to cover a window as a whole once it registered as touched) solved a visual
bug but reintroduced the discarded performance problem for exactly this
reason; the working version only ever redraws the actually-overlapping
sub-region of each window, with no region expansion at all.

## Window chrome and lifecycle

Beyond the base window abstraction, the window system implements the full
set of interactions expected of a real desktop window: draggable and
resizable frames with real title bars, minimize/maximize (with the
original position/size remembered for restore), edge-snapping to half or
full screen, popups/menus that close themselves on an outside click without
also triggering whatever's underneath, and an interceptable close
lifecycle — a window can be *asked* to close without being forced to,
letting an application decide whether to confirm or veto the request. Title
bars, menus, and popups are built from the same widget toolkit described in
[ui.md](ui.md), not one-off drawing code specific to window chrome. Every
one of these mechanisms was built and verified independently, as a
testable unit, before being wired into the live desktop session (see
[desktop.md](desktop.md)).

## Where this shows up later

- The desktop session (see [desktop.md](desktop.md)) is what actually
  drives this compositor and window system continuously, from real
  mouse/keyboard interrupts rather than synthetic test input.
- Icons and other loaded assets (see [graphics.md](graphics.md#the-image--asset-system-m35-in-progress))
  are meant to eventually appear in window chrome (title bars, the
  taskbar) once M35 reaches that point.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M28–M34 range).
