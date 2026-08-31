# 0004 — Image immutability: convention over a type-level split

> 🇧🇷 [Ler em Português](../../pt-br/decisions/0004-image-immutability.md)

**Status**: Decided (Image / Asset System milestone). **Area**: [graphics](../architecture/graphics.md).

## Context

The graphics engine's core abstraction is a `Surface` — anything that can
report its size and read/write individual pixels — implemented by both the
real framebuffer and an in-memory drawing buffer. Introducing a loaded,
effectively-immutable image asset raised the question of whether that
immutability should be a real, type-system-enforced guarantee, or a
documented convention backed by a runtime no-op.

## Alternatives considered

- **(a) Reuse the existing mutable drawing-buffer type directly** as "the"
  image type, with no new type at all. Simplest option, but conflates two
  genuinely different concepts (a window's mutable, redrawn-every-frame
  surface vs. a loaded, reused asset).
- **(b) Split the existing `Surface` trait into a read-only supertrait**
  (size + read-pixel only) and a full read/write trait built on top of it,
  with the new image type implementing only the read-only one — making
  "you cannot write to this" a compile-time guarantee, not a convention.
- **(c) A new image type implementing the existing full `Surface` trait**,
  with its write-pixel method being a deliberate, documented no-op.

## What happened

Alternative (b) was tried first — it looked like the architecturally
"correct" choice, since it would make immutability a real type-system
guarantee rather than a documented promise. Building it revealed the actual
cost: dozens of call sites throughout the codebase that only import the
read/write trait (because that's the only one that existed before) would
each need to additionally import the new read-only trait too, just to keep
resolving methods they already called — Rust's method resolution requires
the specific trait providing a method to be in scope, not just any
supertrait of it. None of those call sites were doing anything wrong; they
would only need the change to keep compiling at all.

## Decision

**(c)** — the new image type implements the existing, full `Surface` trait,
and its write-pixel method is a documented no-op. This meant the existing,
generic "copy a region from one surface to another" operation (already used
throughout the graphics engine) worked with the new type immediately, with
zero changes to its signature and zero changes to any of the dozens of
unrelated call sites that only needed read access in the first place.

## Consequences

- The immutability guarantee is enforced by documentation and test
  coverage (a test that deliberately calls the no-op write and confirms
  nothing changed) rather than by the type system.
- This was judged an acceptable trade-off specifically because no real
  consumer had ever attempted to write to a loaded image by mistake — the
  stronger guarantee would have been solving a problem that didn't
  actually exist yet, at a real cost to every existing call site.
- If a real bug involving accidental writes to a loaded image ever
  surfaces, alternative (b) — or some other type-level enforcement — is
  the natural next thing to revisit, now backed by a concrete case for why
  it's worth the ripple effect.
