# Coding Principles & Methodology

> 🇧🇷 [Ler em Português](../../pt-br/design/coding-principles.md)

This page describes *how* Lynox is built, as a process — the rules that
apply to every milestone, independent of which subsystem it touches. See
[`docs/en/architecture/`](../architecture/overview.md) for what's been
built, [`docs/en/design/ui-guidelines.md`](ui-guidelines.md) and
[`docs/en/design/graphics-guidelines.md`](graphics-guidelines.md) for
subsystem-specific design principles, and
[`docs/en/decisions/`](../decisions/) for specific decisions made under
these rules.

## The per-submilestone discipline

Every submilestone in the roadmap follows the same sequence:

1. **Goal** — what problem is this solving, stated concretely.
2. **Real alternatives** — the actual options considered, not a strawman
   next to the chosen approach. If there's only one reasonable option, that
   gets said explicitly rather than padded out with fake alternatives.
3. **Justified decision** — which option was picked, and why, in terms of
   the actual trade-off (not "it's cleaner" without saying cleaner *how*, or
   at what cost).
4. **Architecture** — the shape of the solution before writing it.
5. **Implementation.**
6. **Verification** — see below; this step is treated as non-negotiable.
7. **Documentation** — the decision, the alternatives rejected, and what was
   actually verified get written down, not just the resulting code.

## Verification means running it, not compiling it

A successful build proves the code is syntactically and type-valid — it
proves nothing about whether the feature actually works. Every milestone is
verified by actually booting the system (typically headless in QEMU,
checked against an expected pass/fail marker on serial output) and, for
anything touching real hardware-facing code, repeated multiple times to
catch intermittent failures a single run would miss. Milestones that touch
interactive behavior (mouse/keyboard input, the live desktop session) have
been verified by hand, interactively, specifically because some classes of
bug only appear under real, sustained, interrupt-driven use — no amount of
synthetic automated testing would have found them (see
[`docs/en/architecture/desktop.md`](../architecture/desktop.md) for a
concrete example).

## No speculative complexity

A recurring rule, phrased throughout the project as "no feature gets
added because it would be cool": every new abstraction needs a concrete
consumer or a real architectural need *at the time it's built*, not a
hypothetical future one. When a design choice would only pay off for a
consumer that doesn't exist yet, the simpler option is chosen now, and the
more general one is deferred until something real actually needs it. This
shows up constantly in smaller decisions too — e.g., choosing the simplest
correct algorithm (nearest-neighbor image scaling, Bresenham line drawing)
over a more sophisticated one, because nothing has demonstrated that the
simpler version's quality is actually a problem yet — see
[`docs/en/design/graphics-guidelines.md`](graphics-guidelines.md) for how
this plays out specifically in the graphics engine.

## Small, isolated changes

Each submilestone is scoped to be independently understandable and
independently verifiable. When an implementation attempt reveals that the
"more correct" design would require touching dozens of unrelated call sites
across the codebase for a benefit nothing currently needs, that's treated as
a signal to step back and pick the smaller, more contained option instead —
see [`docs/en/decisions/`](../decisions/) for a specific example of this
trade-off actually happening.

## Honest documentation of failure

Bugs found during verification — including ones that turned out to be
self-inflicted regressions introduced earlier in the same session — are
recorded in the architecture docs as real findings, with what caused them
and how they were fixed, rather than silently patched and left
undocumented. Documentation is written to be read as an honest account of
what actually happened, not a polished feature list after the fact.

## What this looks like from the outside

The practical result of these rules is a project where every milestone in
[`milestones/en/milestones.md`](../../../milestones/en/milestones.md)
corresponds to a real, verified, documented unit of work, not a curated
highlight reel.
