# Graphics Guidelines

> 🇧🇷 [Ler em Português](../../pt-br/design/graphics-guidelines.md)

Design principles specific to the [graphics engine](../architecture/graphics.md)
and [window system](../architecture/window-system.md), on top of the
general rules in [`coding-principles.md`](coding-principles.md).

## One abstraction, every destination

Drawing code is written once against the
[`Surface` abstraction](../architecture/graphics.md#surfaces-one-abstraction-two-destinations)
and never against a specific concrete destination (the real framebuffer vs.
an offscreen buffer). This is what makes double buffering, compositing, and
testing all possible without duplicating a single drawing algorithm — a
rectangle-fill routine written once works identically whether its output
ever reaches a screen or stays in a test's in-memory buffer.

## Correctness first, performance work is measured, not guessed

New graphics/compositing features are built for correctness first; a
performance regression is only accepted as fixed once it's actually
*measured* to be fixed (a live FPS counter — see
[`docs/en/architecture/desktop.md`](../architecture/desktop.md#rendering-frame-pacing-and-a-live-fps-counter) —
exists specifically so this isn't guesswork). This project's compositor
damage-tracking history (see
[0005 — Damage tracking](../decisions/0005-damage-tracking.md)) is the
concrete cautionary example: a fix that looked like a natural, incremental
improvement over a known bug turned out to silently regress the very
performance property it was supposed to preserve, and that was only caught
because performance was re-measured, not assumed.

## Simplest correct algorithm, upgraded only when something real demands it

Nearest-neighbor image scaling, Bresenham line drawing, per-pixel blitting
— in every case, the simplest algorithm that produces a *correct* result is
chosen over a more sophisticated one, until a real, demonstrated need for
better quality or speed shows up. This is the graphics-specific instance of
the general "no speculative complexity" rule (see
[`coding-principles.md`](coding-principles.md#no-speculative-complexity)):
a smoother scaling filter or a hardware-accelerated copy path is real,
useful work *when something needs it* — building it earlier just means
maintaining more complexity for a benefit nobody has asked for yet.

## Damage, not brute force

Once a scene can change incrementally (windows moving, a cursor animating),
redrawing the entire screen on every frame regardless of what changed is
treated as the thing to avoid, not the default to reach for. Damage
tracking (see
[`docs/en/architecture/window-system.md`](../architecture/window-system.md#damage-tracking))
exists because a static desktop shouldn't cost the same to render as one
where everything is moving at once — but see the note above: a damage-
tracking optimization is not considered "done" until its performance
benefit is actually confirmed, not just its correctness.
