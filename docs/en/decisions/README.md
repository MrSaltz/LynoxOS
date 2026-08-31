# Decisions

> 🇧🇷 [Ler em Português](../../pt-br/decisions/README.md)

Architecture Decision Records for Lynox — specific, point-in-time technical
decisions, written as: context, the real alternatives considered, the
decision, and its consequences. These are a curated subset of the decisions
made across the project (see [`docs/en/architecture/`](../architecture/overview.md)
for how each subsystem works) — chosen because they illustrate a trade-off
or a piece of reasoning that's useful on its own, outside the specific
milestone it came from.

- [0001 — Kernel language: Rust](0001-rust.md)
- [0002 — Bootloader: Limine](0002-limine.md)
- [0003 — Kernel model: hybrid, not monolith or pure microkernel](0003-kernel-model.md)
- [0004 — Image immutability: convention over a type-level split](0004-image-immutability.md)
- [0005 — Compositor damage tracking: partial blit, not region expansion](0005-damage-tracking.md)

See [`docs/en/design/coding-principles.md`](../design/coding-principles.md)
for the general rules these decisions were made under.
