# IPC

> 🇧🇷 [Ler em Português](../../pt-br/architecture/ipc.md)

Covers the inter-process communication primitive introduced in the
Concurrency phase (M11): how separate, isolated processes (see
[processes.md](processes.md)) talk to each other, as distinct from how
threads *within* one process synchronize (see [threads.md](threads.md)).

## Why IPC needed to be designed early, not bolted on

Lynox's kernel model is a hybrid, with microkernel discipline enforced at
the boundaries (see
[0003 — Kernel model](../decisions/0003-kernel-model.md)): a small core
owns CPU, memory, scheduling, and security primitives, while drivers and
services are architected as if they were separate components communicating
over IPC — even the ones currently running in kernel space for bring-up
simplicity. Whether "move this driver to userspace later" ends up being an
easy migration or a full rewrite is decided almost entirely by whether IPC
was designed as a real, central primitive from the start. Retrofitting IPC
after a system has already grown around direct function calls and shared
memory is one of the most common ways a hybrid-kernel project quietly
drifts back into being a monolith in practice, regardless of its stated
architecture.

## Design reference point

Lynox's IPC design takes the handles/channels model used by modern
capability-oriented microkernels (Zircon/Fuchsia is the specific reference
point) as a starting point for evaluation — not to copy it wholesale, but
because that design already solved the specific problem Lynox needs solved:
efficient, capability-respecting communication between isolated processes,
designed as a first-class primitive rather than an add-on. Every IPC
operation is checked against the same capability model the rest of the
syscall surface uses (see [capabilities.md](capabilities.md)) — a process
can only communicate through a channel it actually holds a capability for.

## Where this shows up later

- Drivers and system services that are architecturally separate from the
  kernel core (see [userspace.md](userspace.md)) rely on this layer to
  talk back to the kernel and to each other.
- The capability model (see [capabilities.md](capabilities.md)) is what
  makes an IPC endpoint a controlled resource rather than an ambient
  channel anything can reach.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M11).
