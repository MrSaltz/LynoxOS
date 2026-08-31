# Threads & Concurrency

> 🇧🇷 [Ler em Português](../../pt-br/architecture/threads.md)

Covers the Concurrency phase (M8–M11): turning a kernel that can only run
one thing at a time (see [kernel.md](kernel.md)) into one that can run many
things safely, at once, preemptively.

## The preemptive scheduler

Lynox uses a preemptive scheduler — threads don't have to yield
voluntarily; a timer interrupt (the PIT, programmed early in boot, see
[kernel.md](kernel.md#bring-up-sequence)) forces a context switch on a
regular cadence. Threads are the kernel's basic unit of execution, and the
desktop itself is ultimately just one more scheduled thread (see
[desktop.md](desktop.md)) that happens to never return.

Building a preemptive scheduler correctly, on its own, surfaces real
concurrency bugs early — which is why this phase comes right after
Foundation, before anything that depends on threads not stepping on each
other (drivers, the filesystem, the network stack, the compositor) gets
built.

## Concurrency primitives

On top of the scheduler, Lynox implements the standard toolkit a kernel
needs to let multiple threads (and later, interrupt handlers) touch shared
state safely:

- **Blocking / wakeup** — a thread can suspend itself waiting for a
  condition and be woken by another thread or an interrupt handler, instead
  of busy-spinning.
- **Mutex** — mutual exclusion for kernel data structures.
- **Condvar** — condition variables layered on top of blocking/wakeup, for
  "wait until this specific thing changes" rather than polling a lock.
- **Semaphore** — counting synchronization, used where a mutex's binary
  model doesn't fit (e.g. a bounded pool of resources).
- **RwLock** — reader/writer locking where many concurrent readers are
  common and a writer needs exclusive access.

A recurring theme across the whole project (see
[`docs/en/design/coding-principles.md`](../design/coding-principles.md)) is
that these primitives get *used*, not just implemented — later milestones
(the AHCI driver's command queue, the TCP/IP stack, the compositor, the
desktop session) exercise them under real, concurrent, interrupt-driven
load, which is where subtle bugs (deadlocks, lost wakeups, priority
inversion) actually show up. The live desktop session (see
[desktop.md](desktop.md)) found a real deadlock this way — closing an
application from inside a lock it still needed to release — that no
synthetic single-threaded test had ever exercised.

## Where this shows up later

- Every driver, the [userspace layer](userspace.md), the
  [compositor](window-system.md), and the
  [live desktop session](desktop.md) run as scheduled threads and rely on
  these primitives to touch shared state safely.
- [IPC](ipc.md) is a related but distinct concern: these primitives
  synchronize threads *within* a shared address space, while IPC is about
  communication *between* separate, isolated processes.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M8–M11).
