# Lynox Documentation

> 🇧🇷 [Ler em Português](../pt-br/README.md)

This is the documentation hub — the full landing page lives at the
[repository root](../../README.md). This section indexes the deeper
material: how each subsystem works, the principles behind the design, and
specific engineering decisions.

## Architecture

How and why each subsystem works the way it does — see
[`architecture/overview.md`](architecture/overview.md) for the map. Covers:
[kernel core](architecture/kernel.md), [userspace](architecture/userspace.md),
[memory](architecture/memory.md), [processes](architecture/processes.md),
[threads](architecture/threads.md), [capabilities](architecture/capabilities.md),
[IPC](architecture/ipc.md), [filesystem](architecture/filesystem.md),
[graphics](architecture/graphics.md), [window system](architecture/window-system.md),
[UI toolkit](architecture/ui.md), and [desktop](architecture/desktop.md).

## Design

The principles behind the engineering process and the subsystem-specific
design rules:

- [`design/coding-principles.md`](design/coding-principles.md) — the
  per-submilestone methodology and the general rules (no speculative
  complexity, verification means running it, honest documentation of
  failure).
- [`design/ui-guidelines.md`](design/ui-guidelines.md) — design principles
  for the UI toolkit and window system.
- [`design/graphics-guidelines.md`](design/graphics-guidelines.md) — design
  principles for the graphics engine and compositor.

## Decisions

[`decisions/`](decisions/) — Architecture Decision Records: specific,
point-in-time technical decisions, with the real alternatives considered
and the consequences accepted.

## Elsewhere in this repository

- [`roadmap/en/roadmap.md`](../../roadmap/en/roadmap.md) — direction and
  phases.
- [`milestones/en/milestones.md`](../../milestones/en/milestones.md) — the
  full milestone checklist.
