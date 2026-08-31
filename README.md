# Lynox

> 🇧🇷 [Ler em Português](docs/pt-br/README.md)

**A UEFI x86-64 operating system built from scratch in Rust** — kernel, drivers,
virtual memory, processes/threads, capability-based syscalls, a persistent
filesystem, a TCP/IP stack, an ELF loader, a 2D graphics engine, a windowing
system, a compositor, and a desktop with its own applications.

The Lynox kernel and operating-system architecture are developed from
scratch in Rust. The project does not derive from Linux, Windows, or another
existing operating-system codebase. Third-party dependencies and standards
are used where explicitly documented (see
[Third-party components](#third-party-components)).

> **This repository is the public engineering log of Lynox — not the kernel
> source.** The kernel, drivers, and userspace code are developed in a
> private repository while the project is still under active, early-stage
> development. What's here is real: the roadmap, the methodology, a
> commit-by-commit changelog of the actual engineering work, and a set of
> architecture write-ups that explain *how* and *why* each subsystem works
> the way it does, without walking through the private implementation line
> by line. See [Public vs. private](#public-vs-private) below.

## Status

**M0–M34 complete, M35 (Image / Asset System) in progress (M35.1–M35.6
done)** — ~265 commits. Foundation, concurrency, drivers/PCI/storage/VFS,
TCP/IP networking, virtual memory, processes, capability syscalls, kernel and
userspace runtimes, an ELF loader, and a graphical desktop are all done.
**M34 (Advanced Window System)** is fully closed — real window frames,
resize, drag, minimize/maximize, modal windows, popups/menus, multi-window
applications, an interceptable close lifecycle, edge snapping, size
constraints, resize handles, focus-aware decorations, always-on-top/modal
render ordering, a clickable taskbar, and a **persistent, truly interactive
live desktop session** driven by real mouse/keyboard IRQs (not synthetic test
packets), with a real-time FPS counter and a damage-aware compositor. **M35
(Image / Asset System)** is underway: a loaded, immutable image type, pixel
format conversion, loading raw bytes from the filesystem, decoding them into
a usable image, and nearest-neighbor scaling are all done.

- Roadmap and direction: [`roadmap/en/roadmap.md`](roadmap/en/roadmap.md)
- Milestone checklist: [`milestones/en/milestones.md`](milestones/en/milestones.md)
- Architecture write-ups (how and why): [`docs/en/architecture/overview.md`](docs/en/architecture/overview.md)
- Design principles and methodology: [`docs/en/design/coding-principles.md`](docs/en/design/coding-principles.md)
- Notable engineering decisions: [`docs/en/decisions/`](docs/en/decisions/)

## Highlights

- **Own kernel in Rust** (`no_std`, bare-metal) — no dependency on a host
  libc/OS.
- **Full virtual memory**: custom paging, copy-on-write, real process
  isolation via CR3 switching.
- **Preemptive scheduler** with threads, blocking/wakeup, mutex/condvar/
  semaphore/rwlock.
- **Capability-based syscalls** (not Unix-style uid/gid) — a process can only
  touch a resource it holds an explicit capability for.
- **Own persistent filesystem** over a real AHCI driver (DMA, command
  list/FIS/PRDT), with an LRU write-through cache.
- **Own TCP/IP stack** over a real e1000 driver: Ethernet/ARP/IPv4/UDP/
  TCP/Sockets/DHCP/DNS/IPv6, verified over loopback and emulated hardware.
- **Real ELF loader**: parsing, segment mapping, relocations, ABI
  validation — userspace programs are compiled normally, not hand-written
  bytes.
- **Own 2D graphics engine** over the UEFI framebuffer (GOP): surfaces,
  primitives, fonts, a compositor with double buffering and damage tracking.
- **Working desktop**: 5 applications (File Manager, Settings, System UI,
  Terminal, Taskbar) on top of an in-house UI toolkit, with a real PS/2 mouse
  driver routing clicks to the right window and a clickable taskbar.
- **Full window management**: resizable, draggable, minimizable/
  maximizable, modal, and always-on-top windows, with edge snapping, size
  constraints, resize handles, and focus-reactive decorations.
- **Live, persistent desktop session**: a scheduler thread that never
  returns, consuming real mouse/keyboard interrupts and rendering the result
  continuously with a damage-aware compositor and a live FPS counter —
  verified interactively across two independent hypervisors (QEMU and
  VirtualBox).

## Methodology

Every submilestone follows the same discipline: goal → real alternatives
considered → justified decision → architecture → implementation →
verification → documentation. Verification means actually *running* the
result (a headless QEMU boot checked for a pass/fail marker on serial
output, repeated multiple times for stability), never just a successful
compile. Bugs found along the way are recorded honestly in the changelog and
the architecture write-ups — including the ones that turned out to be
self-inflicted regressions — instead of being silently patched away. See
[`docs/en/design/coding-principles.md`](docs/en/design/coding-principles.md)
for the full set of guiding rules (no premature abstraction, no speculative
complexity, a real consumer required before a feature is built).

## Public vs. private

Lynox is built in the open in the sense that matters most to an engineering
audience: the roadmap, the reasoning behind every design decision, the full
commit history and what each commit changed, and (as demos become available)
the system actually running are all here. What stays private for now is the
literal kernel/driver/userspace source code, because the project is still in
active, early-stage development, and the intent is to keep the engineering
log open without opening code review on a moving target before it's ready.

This is not a promise that the code will *never* be published, and it's not
an attempt to hide that private parts exist — the architecture docs describe
every subsystem, including the ones whose source isn't here. It's a
deliberate, current-stage choice about what's ready to be reviewed as code
versus what's ready to be read as an engineering story.

## Third-party components

- **Limine** — UEFI/BIOS bootloader (binary, vendored, not modified).
- **OVMF (TianoCore)** — UEFI firmware used for development/testing in QEMU.
- A small number of Rust crates for low-level primitives (e.g. CPU/register
  bindings, a bitmap font for the console). The kernel, drivers, filesystem,
  network stack, graphics engine, windowing system, and desktop are original
  code.

## Roadmap

- [`roadmap/en/roadmap.md`](roadmap/en/roadmap.md) — direction, phases, and what's not in scope.
- [`milestones/en/milestones.md`](milestones/en/milestones.md) — one line per top-level milestone (M0–M65), what's done and what's left.

Phase structure (see [`milestones/en/milestones.md`](milestones/en/milestones.md) for the full list):
Foundation (M0–M7) → Concurrency (M8–M11) → Hardware/Drivers/Networking
(M12–M17) → Memory/Processes/Userspace (M18–M23) → Runtime/Executables
(M24–M26) → Graphics/Desktop (M27–M32) → **Ecosystem (M33–M42, in progress
— M34 closed, M35 in progress)** → SMP/Advanced Hardware (M43–M47) →
Security/Reliability (M48–M54) → Platform (M55–M64) → Final Release (M65).

## Screenshots / demos

[`screenshots/`](screenshots/) and [`demos/`](demos/) will hold captures as
milestones produce something visually worth showing (the live desktop
session, the windowing system, real applications) — organized by boot,
desktop, applications, and development. Empty for now.

## Following the project / feedback

Issues are open for questions, feedback, and discussion about the
architecture or roadmap. Code contributions to the kernel aren't being
accepted while the source stays private — see
[`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md) for the current stance
and what kind of Issues are welcome, and
[`.github/SECURITY.md`](.github/SECURITY.md) for how to report a security
concern.

## License

See [`LICENSE`](LICENSE). It covers the content of this repository
(documentation, architecture write-ups, roadmap, changelog) — it does not
grant any rights to the private Lynox kernel/OS source code, which is not
included here.

"Lynox" is used here only as this project's name. It is not asserted as a
registered or claimed trademark, and no affiliation with, or endorsement
by, any other product, company, or trademark of a similar name is intended
or implied.
