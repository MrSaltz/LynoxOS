# 0002 — Bootloader: Limine

> 🇧🇷 [Ler em Português](../../pt-br/decisions/0002-limine.md)

**Status**: Decided (foundational, before M0). **Area**: [overview](../architecture/overview.md#boot-flow).

## Context

A UEFI x86-64 kernel needs something to leave firmware Boot Services, set
up an initial environment (a memory map, a framebuffer, an ACPI pointer,
basic paging), and jump to the kernel's entry point already in long mode.
Writing that stage from scratch, just to prove the kernel itself works,
would mean solving a mostly-solved problem before getting to write any
actual kernel code.

## Alternatives considered

- **Write a custom UEFI bootloader.** Full control, but a large amount of
  work (UEFI protocol handling, GOP negotiation, memory map retrieval, ACPI
  discovery) spent before the kernel proper does anything — the wrong place
  to spend early risk budget for this project.
- **`bootimage`/hand-rolled BIOS boot.** Would drag in legacy BIOS concerns
  the project has no interest in supporting (see
  [`roadmap`](../../../roadmap/en/roadmap.md) — UEFI-only is explicit
  scope).
- **Limine.** A mature, actively maintained UEFI+BIOS bootloader used by
  many hobby OS projects and some larger ones. It delivers a memory map, a
  GOP framebuffer, and an ACPI RSDP pointer to the kernel in a well-defined
  protocol, and ships prebuilt binaries that don't need to be compiled as
  part of this project's own build.

## Decision

**Limine**, using its prebuilt UEFI binaries (vendored, not modified), with
a UEFI-only ISO (no BIOS boot entry — consistent with the project's
declared scope).

## Consequences

- The kernel's very first verifiable milestone (M1 — bootloader hand-off)
  was reachable quickly, because the hard parts of UEFI boot were already
  solved by a mature, external project.
- The project depends on Limine's protocol and binary compatibility going
  forward; this is treated as an acceptable, revisitable dependency — if
  Secure Boot chain-of-trust control ever becomes a real requirement (see
  the security/reliability phase in [`roadmap`](../../../roadmap/en/roadmap.md)),
  this decision may need to be revisited, since it would require the
  project to control more of its own boot chain than a prebuilt bootloader
  binary allows.
