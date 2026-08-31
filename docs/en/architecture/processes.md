# Processes

> 🇧🇷 [Ler em Português](../../pt-br/architecture/processes.md)

Covers the process half of Memory/Processes/Userspace (M18–M23) and the
executable-loading half of the Runtime/Executables phase (M24–M26): turning
an isolated address space (see [memory.md](memory.md)) into something that
can actually run a program. For the runtime layer *on top of* a running
process (syscalls, drivers, filesystem, networking), see
[userspace.md](userspace.md).

## Process lifecycle

A process in Lynox owns: its own address space (see
[memory.md#process-isolation-via-cr3](memory.md#process-isolation-via-cr3)),
one or more threads (see [threads.md](threads.md))
scheduled within that address space, a table of capabilities it holds (see
[capabilities.md](capabilities.md)), and open file descriptors into the
filesystem. Creating, running, and tearing down a process touches all three
of the kernel's core subsystems at once, which is why this phase comes
after memory and concurrency are both solid.

## The ELF loader

Rather than accepting hand-written machine code as the only way to get a
program into userspace, Lynox implements a real ELF64 loader: parsing the
ELF header and program headers, mapping each `PT_LOAD` segment into the new
process's address space with the correct permissions (read-only for
`.rodata`, executable for `.text`, writable for `.data`/`.bss`), and
performing the relocations the binary requires. ABI validation rejects
anything that doesn't match the kernel's expectations (wrong architecture,
wrong class, unsupported segment types) rather than attempting to run
something malformed and hoping for the best.

This matters beyond correctness: it means userspace programs for Lynox are
*compiled normally* — with a standard Rust or C toolchain targeting the
right ABI — rather than requiring a special hand-assembled format. Anything
that can produce a conforming ELF64 binary is a valid Lynox userspace
program, which is what lets the userspace runtime (see
[userspace.md](userspace.md)) exist as ordinary compiled code rather than a
pile of hex bytes embedded in the kernel.

## Where this shows up later

- The desktop's applications (File Manager, Settings, Terminal, System UI —
  see [desktop.md](desktop.md)) are themselves processes running through
  this same lifecycle and loader, not special-cased kernel code.
- Every syscall a process makes is checked against the capability model
  (see [capabilities.md](capabilities.md)) before it's allowed to touch a
  resource.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M18–M26 range).
