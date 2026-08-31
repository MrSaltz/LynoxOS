# Lynox — Roadmap

> 🇧🇷 [Ler em Português](../pt-br/roadmap.md) · [← Back to README](../../README.md)

High-level direction of the project. For the full checklist see
[`milestones/en/milestones.md`](../../milestones/en/milestones.md); for the
architecture write-ups and engineering decisions behind these phases, see
the docs linked from the main [README](../../README.md).

## Where the project is going

Lynox is a from-scratch UEFI x86-64 operating system: its own kernel,
drivers, memory management, networking stack, and a graphical desktop —
not a Linux distro, not a Windows mod, and not built around an existing
compiler or language. It follows a strict, test-verified methodology: every
piece of functionality is designed, implemented, and proven in QEMU before
being marked done.

## Phases

1. **Foundation** — toolchain, boot, CPU init, console, interrupts, physical
   memory, heap, scheduler/threads. ✅ Done
2. **Concurrency** — synchronization primitives, blocking/wakeup, IPC,
   advanced concurrency (condvars, semaphores, rwlocks). ✅ Done
3. **Hardware / Drivers** — driver architecture, PCI enumeration, storage,
   a VFS with a persistent filesystem, device management, a full TCP/IP
   stack. ✅ Done
4. **Memory / Processes / Userspace** — virtual memory, processes, capability
   syscalls, userspace, process lifecycle, security/capabilities. ✅ Done
5. **Runtime / Executables** — kernel and user runtimes, an ELF loader
   compatible with normal toolchains (LLVM/Clang, GCC, Rust). ✅ Done
6. **Graphics / Desktop** — a 2D graphics engine, windowing, a compositor,
   a working desktop with real applications, mouse input, window chrome.
   ✅ Done
7. **Ecosystem** *(current phase)* — deep UI interaction, an advanced
   window system, an SDK, an application model, developer tools. 🚧 In
   progress — see below.
8. **SMP / Advanced Hardware** — multi-core support, advanced interrupt
   controllers, PCIe features. ⬜ Planned
9. **Security / Reliability** — hardening, sandboxing, fault handling,
   release engineering. ⬜ Planned
10. **Platform** — audio, physical hardware compatibility, a production
    toolchain, an application platform, network services, graphics
    acceleration, storage expansion, installation/recovery, POSIX
    compatibility, system services. ⬜ Planned
11. **Final Release** — Lynox 1.0. ⬜ Planned

## Current focus: Ecosystem phase

- **UI Interaction** — keyboard focus, double-click, pointer capture,
  unified input handling. ✅ Done
- **Advanced Window System** — real window frames, resize, drag, minimize/
  maximize, modal windows, popups/menus, multi-window applications, an
  interceptable close lifecycle, edge snapping, size constraints, resize
  handles, focus-reactive decorations, consistent always-on-top/modal
  rendering, and a persistent, truly interactive live desktop session
  driven by real mouse/keyboard interrupts. ✅ Done
- **Image/Asset System** — a loaded, immutable image type distinct from a
  window's mutable surface. 🚧 In progress
- **Rendering Pipeline, UI Composition, SDK, Application Model, Package
  Manager, Developer Tools, Compatibility Layer** — planned next, in that
  order. ⬜

## What this project is *not*

- Not a custom Linux distribution or a modification of an existing kernel.
- Not building its own programming language or compiler — Lynox targets
  existing toolchains (LLVM/Clang, GCC, Rust).
- Not a game engine, and never will host one as part of the OS roadmap.

## Guiding principles

- Hardware access always goes through a real abstraction layer (driver →
  generic abstraction → kernel), never hardcoded to one virtualization
  platform.
- No feature is added "because it'd be cool" — every new milestone needs a
  concrete consumer or architectural need.
- Completed milestones and their history are never reopened or rewritten
  without a proven technical need.
