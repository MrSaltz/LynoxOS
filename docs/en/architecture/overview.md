# Architecture Overview

> 🇧🇷 [Ler em Português](../../pt-br/architecture/overview.md)

This page is the map. It explains the foundational choices that shape every
other subsystem, and links to a dedicated write-up for each major area.
None of these pages walk through private source code line by line — they
explain the *shape* of each subsystem and the reasoning behind it, at the
level a systems engineer would want before deciding whether the design
holds up.

- [Kernel core](kernel.md) — boot, CPU bring-up, interrupts, physical
  memory.
- [Userspace](userspace.md) — drivers, the TCP/IP stack, the userspace
  runtime.
- [Memory](memory.md) — virtual memory, paging, copy-on-write, process
  isolation.
- [Processes](processes.md) — process lifecycle, the ELF loader.
- [Threads](threads.md) — the preemptive scheduler, concurrency primitives.
- [Capabilities](capabilities.md) — the capability-based syscall model and
  why it isn't Unix-style uid/gid.
- [IPC](ipc.md) — inter-process communication between isolated processes.
- [Filesystem](filesystem.md) — the AHCI driver and the Virtual Filesystem.
- [Graphics](graphics.md) — the 2D graphics engine, surfaces, the image/
  asset system.
- [Window system](window-system.md) — windowing, chrome, the compositor.
- [UI toolkit](ui.md) — the widget toolkit and interaction model.
- [Desktop](desktop.md) — the live desktop session, input routing, the
  built-in applications.

## Kernel model: hybrid, with microkernel discipline at the boundaries

Three real options exist for how much lives in kernel space: a pure
monolithic kernel (Linux), a pure microkernel (seL4, MINIX 3), or a hybrid
(Windows NT, XNU). A pure monolith is the fastest way to get something
working, but retrofitting isolation later is painful — a bug in one driver
can take down the whole kernel. A pure microkernel gives the strongest
isolation, but building one from scratch, alone, is a multi-year project
just to get the core right, and a naively-designed IPC layer can dominate
performance.

Lynox takes the hybrid path deliberately: a small core owns CPU, memory, the
scheduler, interrupts, and the security primitives (capabilities). Drivers
and system services are architected as if they were separate, IPC-speaking
components from day one — even the ones that currently run in kernel space
for bring-up simplicity use the same interface a userspace version would.
That's the detail that matters: the goal isn't "monolithic now, microkernel
later" (that migration essentially never happens in practice, because
nothing was designed for it) — it's "hybrid now, with the seams already cut
in the right places."

## Language: Rust

The kernel language is the most expensive decision in the project to
reverse — it determines the toolchain, what's easy to write, and the shape
of every bug class that's possible. Lynox picked Rust over C (the historical
default for OS dev) because the project's first priority is security, and
memory safety by construction eliminates the most common bug class in C
kernels (buffer overflows, use-after-free, data races) without depending on
perfect human discipline sustained over years. The bare-metal Rust ecosystem
was mature enough by the time this decision was made — real crates for
CPU/register access, a maintained bootloader ecosystem, and a real
reference point in production use (a full Redox OS exists) — that the extra
learning curve (fighting the borrow checker in `unsafe`/`no_std` code while
also learning OS development) was judged worth it.

Assembly (NASM) is used for the minimum unavoidable pieces — the boot entry
stub, context switching, interrupt stubs. C is allowed only as a pointed
exception if a low-level library with no Rust equivalent ever needs
vendoring; that hasn't been necessary so far.

## Toolchain

- **Build host**: WSL2 (Ubuntu) — OS-dev tooling (`nasm`, `qemu`, `ovmf`,
  `gdb`) is a first-class citizen on Linux; this avoids fighting Windows
  path/permission quirks for no benefit.
- **Rust toolchain**: `rustup` nightly (required for `no_std`/`build-std`
  features), custom target `x86_64-unknown-none` — a bare-metal kernel
  can't use a host's precompiled `std`.
- **Linker**: `lld` (ships with Rust/LLVM) plus a custom linker script, for
  full control over the kernel's memory layout.
- **Bootloader**: [Limine](https://github.com/limine-bootloader/limine) — a
  mature, actively maintained UEFI/BIOS bootloader. It hands the kernel a
  memory map, a GOP framebuffer, and an ACPI RSDP pointer, which avoids
  reinventing the boot stage just to prove the kernel can run.
- **VM firmware**: OVMF (TianoCore) — the standard open-source UEFI
  firmware for QEMU.
- **Emulator**: QEMU, for fast iteration, snapshotting, and GDB attachment
  without needing physical hardware every cycle.

## Boot flow

```
Power on
  → UEFI firmware (OVMF in the VM / real UEFI on physical hardware)
  → Limine (loads kernel.elf, builds the memory map, GOP framebuffer, ACPI RSDP)
  → Limine exits UEFI Boot Services and jumps to the kernel entry point
     (already in long mode, basic paging set up by the Limine protocol)
  → Kernel: builds its own GDT/IDT, brings up the serial console
  → Kernel initializes subsystems in dependency order (memory → scheduler →
     drivers → filesystem → networking → graphics → desktop)
  → The live desktop session starts and never returns
```

Serial output (COM1) was the very first thing the kernel could do, before a
framebuffer console existed — it's the simplest possible way to prove the
kernel is alive, with no dependency on a bitmap font or a video driver. That
ordering (prove it boots, then prove it can draw text) is representative of
how the whole project is sequenced: the riskiest, most foundational claim
gets proven first, every time.

## Executable format

The kernel is a plain ELF64 binary — the native output of the
`x86_64-unknown-none` Rust target, well-supported by Limine and by every
standard debugging tool (`gdb`, `objdump`, `readelf`). There was no reason
to invent a custom format at this stage.

## How this maps to the roadmap

Each of the pages linked above corresponds to one or more milestone ranges
in [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) —
see that file for the full, granular list. See
[`docs/en/design/coding-principles.md`](../design/coding-principles.md)
for the methodology every milestone follows.
