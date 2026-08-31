# Desktop

> 🇧🇷 [Ler em Português](../../pt-br/architecture/desktop.md)

Covers mouse/pointer input (M31) and the live desktop session
(M34.15–M34.16): the layer that turns the window system (see
[window-system.md](window-system.md)) and the [UI toolkit](ui.md) from a
set of independently-testable mechanisms into something a person actually
uses, continuously, driven by real hardware.

## From synthetic tests to a live session

For most of the project's history, the graphical stack was proven correct
through automated tests: a test function feeds a synthetic mouse packet or
key event directly into the code under test and checks the result. That's
the right way to verify each mechanism in isolation, but it's a
fundamentally different thing from a kernel that stays running,
continuously reacting to whatever a real mouse and keyboard actually send
it. M34.15 is the milestone that made that jump: a scheduler thread (see
[threads.md](threads.md)) that never returns, consuming
real interrupt-driven mouse and keyboard input and rendering the result
continuously, instead of a kernel that runs a test suite and goes idle.

That jump surfaced real bugs that synthetic tests structurally couldn't have
found — because they only exist when input arrives at real hardware speed,
interleaved with real interrupts, sustained over time, rather than in a
single controlled function call. Examples: a deadlock that only occurs when
closing an application from within the same lock it needs to release; mouse
packet framing that silently desyncs because of a single unexpected
"acknowledgement" byte that leaks in right as an interrupt line gets enabled
(not observable unless something is actually listening on that IRQ when it
happens); and the cursor never being drawn on screen at all, despite every
other piece of mouse handling working, simply because nothing in the
automated tests had ever needed to *look* at the screen. Verifying this
milestone required abandoning headless automated regression in favor of
manual, interactive testing — and cross-checking behavior across two
independent hypervisors (QEMU and VirtualBox), specifically to separate a
genuine kernel bug from an artifact of one particular virtualization
environment.

## Input routing

Mouse input comes from a real PS/2 driver: packets are decoded, cursor
position is tracked, and clicks are routed through the window system's
z-order (see [window-system.md](window-system.md#the-compositor-and-z-order))
to determine which window — and, from there, which widget within that
window (see [ui.md](ui.md#interaction-model)) — a click or key event
actually lands on. The desktop session owns this routing; the widget-level
focus/double-click/pointer-capture behavior itself lives in the UI toolkit.

## Rendering: frame pacing and a live FPS counter

The render loop was originally reactive — recompose the screen only when
something reported activity. That model produced visible correctness bugs
under sustained interactive use (elements briefly disappearing depending on
mouse position) once damage tracking was introduced (see
[window-system.md](window-system.md#damage-tracking)), and was replaced with
a steady, frame-paced loop: input is still processed every iteration, but
rendering is paced to a fixed interval, decoupling "how fast we react to
input" from "how fast we redraw." A real, live-sampled FPS counter reports
the actual achieved frame rate — software rendering on an emulated,
unaccelerated framebuffer currently sustains roughly 15–20 FPS, which is
accepted as a known, documented limitation of this stage rather than
something to keep chasing; a real improvement here depends on hardware-
accelerated or driver-level rendering work planned for a later milestone.

## Applications and the taskbar

Five built-in applications run on top of the desktop today — a File
Manager, a Settings app, a System UI (showing live stats, including the FPS
counter above), a Terminal, and the Taskbar itself — all built on the same
[UI toolkit](ui.md) and the same [window system](window-system.md) every
other window uses; none of them are special-cased inside the kernel. The
taskbar entries are clickable: clicking one restores/focuses the
corresponding window, reusing the exact same layout logic a generic UI
panel uses internally, rather than a separate hand-rolled layout just for
the taskbar.

## Where this shows up later

- Loaded icons (see [graphics.md](graphics.md#the-image--asset-system-m35-in-progress))
  are intended to eventually replace placeholder visuals in the taskbar and
  window title bars.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M31, M33, M34 range).
