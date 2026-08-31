# Memory

> 🇧🇷 [Ler em Português](../../pt-br/architecture/memory.md)

Covers the memory half of the Memory/Processes/Userspace phase (M18–M23):
turning the physical frame allocator (see [kernel.md](kernel.md#bring-up-sequence))
into full virtual memory with real process isolation.

## Why virtual memory, and why now

Everything before this phase runs in one flat physical address space — fine
for a single-threaded kernel proving out drivers and concurrency primitives,
but incompatible with the project's stated priority of strong isolation
(see [overview.md](overview.md#kernel-model-hybrid-with-microkernel-discipline-at-the-boundaries)).
Real process isolation requires each process to believe it owns the entire
address space, with the CPU's paging hardware enforcing that belief.

## Paging

Lynox implements x86-64's 4-level paging (PML4 → PDPT → PD → PT) directly,
rather than depending on a paging abstraction crate, so the kernel has full
control over page table layout, permission bits, and how kernel-space and
user-space mappings coexist in the same page tables (kernel mappings are
present in every process's address space at the appropriate privilege
level, so a syscall or interrupt doesn't need an address space switch to
reach kernel code).

## Copy-on-write

Rather than deep-copying a process's entire address space on every `fork`-
style operation, pages are shared read-only between parent and child until
one of them writes — at which point a page fault triggers an actual copy,
and only that one page. This is the standard technique for making process
creation cheap without giving up isolation: the CPU's own page fault
mechanism is what enforces "you can share until you diverge."

## Process isolation via CR3

Each process gets its own top-level page table (PML4), and a context switch
between processes with different address spaces reloads the CR3 register to
point at the new process's page tables. This is what makes process isolation
real rather than a convention two processes agree to respect: a process
literally cannot construct a pointer that reaches another process's private
memory, because the hardware won't translate it.

## Where this shows up later

- Process creation and the ELF loader (see [processes.md](processes.md))
  depend on being able to build a fresh address space per process cheaply.
- The capability model (see [capabilities.md](capabilities.md)) is
  meaningful specifically because address-space isolation is real — a
  capability controls access to a *resource*, and process isolation is what
  guarantees a process can't route around that control by reaching into
  another process's memory directly.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M18–M20 range).
