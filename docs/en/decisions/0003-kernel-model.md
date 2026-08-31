# 0003 — Kernel model: hybrid, not monolith or pure microkernel

> 🇧🇷 [Ler em Português](../../pt-br/decisions/0003-kernel-model.md)

**Status**: Decided (foundational, before M0). **Area**: [kernel core](../architecture/kernel.md).

## Context

The project's stated first priority is security — specifically, strong
isolation between drivers/services and the rest of the system, with the
ability to move a component from kernel space to user space later without
a full rewrite. The kernel model decision needed to be made before any code
was written, since it shapes the toolchain, the IPC design, and every
subsystem built on top of it.

## Alternatives considered

| Model | Reference examples | Upside | Downside |
|---|---|---|---|
| Pure monolithic | Linux | Fast to get working, no IPC overhead | Weak isolation by default; retrofitting security later is painful; a driver bug can take down the whole kernel |
| Pure microkernel | seL4, MINIX 3 | Maximum isolation, minimal kernel attack surface, formally verifiable in seL4's case | High IPC cost if designed carelessly; a much steeper build curve; building one from scratch alone is a multi-year project for the core alone |
| Hybrid | Windows NT, XNU (macOS/iOS) | A small kernel plus user-space services where it pays off, while keeping the freedom to keep hot paths in-kernel | Less "pure" in theory; requires discipline to avoid drifting back into a monolith out of convenience |
| Modern pragmatic microkernel | Zircon (Fuchsia) | IPC designed as a first-class primitive (handles, channels) from day one, capability-based from the start | Still a large project; requires treating IPC as central, not an afterthought |

## Decision

**Hybrid, with microkernel discipline enforced at the boundaries.** A small
core owns CPU bring-up, memory, the scheduler, interrupts, IPC, and the
capability-based security primitives. Drivers and system services are
architected as if they were separate, IPC-speaking components regardless of
where they currently execute — some simple drivers currently run in kernel
space for bring-up simplicity, but behind the same interface a user-space
version of the same driver would use.

## Consequences

- This avoids the "microkernel tax" of designing IPC three times (once
  naively, then twice more to fix it), because IPC is designed as a real
  primitive from the point drivers first need it (see [ipc.md](../architecture/ipc.md)),
  not bolted on after the fact.
- It also avoids the common trap where a monolith's "we can isolate this
  later" never actually happens, because nothing was designed with that
  boundary in mind — the interface discipline is enforced *now*, while it's
  still cheap to enforce.
- The explicit trade-off accepted: some kernel-space code today is not
  literally isolated the way a fully microkernel design would enforce.
  That's treated as a deliberate, temporary bring-up simplification, not the
  final architecture — the seams needed to move it to user space later are
  already in place.
