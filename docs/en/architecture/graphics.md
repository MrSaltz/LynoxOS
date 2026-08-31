# Graphics

> 🇧🇷 [Ler em Português](../../pt-br/architecture/graphics.md)

Covers the Graphics/Desktop phase's rendering foundation (M27) and the
in-progress Image/Asset System (M35): everything below the window manager
(see [window-system.md](window-system.md)) that's responsible for turning
pixel data into something on screen.

## Starting point: the framebuffer

The UEFI GOP (Graphics Output Protocol) framebuffer, handed to the kernel
by Limine at boot (see [overview.md](overview.md#boot-flow)), is the one
real destination for pixels on the actual hardware. Everything in the
graphics engine is designed so that this framebuffer is just *one* possible
destination among several — the same drawing code that targets the physical
screen also targets an in-memory buffer, which is what makes double
buffering and a compositor possible at all (see below).

## Surfaces: one abstraction, two destinations

A `Surface` is anything that can report its own size and answer "what color
is the pixel at (x, y)" / "set the pixel at (x, y) to this color." The
physical framebuffer implements it; an off-screen, purely in-memory pixel
buffer also implements it. Every drawing primitive (filled/outlined
rectangles, lines, text, copying a rectangular region from one surface to
another) is written once, generically, against this abstraction — it has no
idea whether it's drawing to the real screen or to a buffer that hasn't
been shown to anyone yet.

That generic "copy a region from one surface to another" operation (a
*blit*) is the single mechanism every higher-level piece of the graphics
stack builds on: presenting a finished off-screen frame to the real
framebuffer is a blit; the compositor drawing a window's contents onto the
final frame is a blit; scaling and drawing a loaded image (below) reuses the
exact same operation with zero changes to its signature.

## Why this split matters: double buffering and damage tracking

Because a `Surface` doesn't care what it represents, the system can draw an
entire frame into an off-screen buffer first and only blit the finished
result to the real framebuffer once — avoiding the visible tearing/flicker
of drawing incrementally straight to the screen. Layered on top of that,
the compositor tracks which *regions* of the screen actually changed between
frames (see [window-system.md](window-system.md#damage-tracking)) so a
static desktop doesn't pay the cost of redrawing everything that didn't
change.

## The Image / Asset System (M35, in progress)

Everything above treats a `Surface` as something that's redrawn every
frame — the right model for a window's contents, wrong for a loaded asset
like an icon, which is decoded once and then reused unchanged, possibly in
several places at once (the same icon in a taskbar entry and in a window's
title bar, for instance).

M35 introduces `Image`: a loaded, effectively-immutable asset, distinct from
a window's mutable drawing surface, but deliberately implementing the same
`Surface` abstraction — so every existing drawing primitive (blitting, in
particular) already works with it, with zero new code needed for that part.
An early design alternative would have split `Surface` into a separate
read-only trait specifically to make an image's immutability enforced by
the type system rather than by convention; it was rejected once it became
clear the ripple effect (every existing call site across the codebase that
only imports the mutable trait would need to import the new one too, just
to keep resolving methods it already used) cost far more than the benefit,
since nothing had ever actually tried to write to an image by mistake. That
kind of trade-off — a theoretically cleaner design versus one that doesn't
require touching dozens of unrelated call sites — comes up often enough in
this project that it has its own write-up: see
[0004 — Image immutability](../decisions/0004-image-immutability.md).

As of M35.1–M35.6: the `Image` type exists, pixel format conversion (RGB/
BGR, with and without an alpha byte) between raw bytes and the engine's
internal color representation is implemented, raw bytes can be loaded from
the [filesystem](filesystem.md) and decoded into a usable `Image`, and
nearest-neighbor scaling produces a resized copy of any surface.
Transparency/alpha compositing, an icon/asset pipeline, and a decode cache
are the next planned steps — see
[`roadmap/en/roadmap.md`](../../../roadmap/en/roadmap.md) and
[`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for
the full list.

## Where this shows up later

- The window system's compositor (see [window-system.md](window-system.md))
  is the main consumer of surfaces, blitting, and damage tracking.
- The [UI toolkit](ui.md) and the desktop's applications (see
  [desktop.md](desktop.md)) draw through this same primitive layer.
