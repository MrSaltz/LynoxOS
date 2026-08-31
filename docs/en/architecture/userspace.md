# Userspace

> 🇧🇷 [Ler em Português](../../pt-br/architecture/userspace.md)

Covers the driver/networking half of the Hardware/Drivers/Networking phase
(M12–M17) and the runtime half of Runtime/Executables (M24–M26): the layer
between a running process (see [processes.md](processes.md)) and the real
hardware/services it depends on. Storage and the filesystem have their own
page — see [filesystem.md](filesystem.md).

## Driver architecture

Drivers in Lynox are written against the same discipline described in the
kernel model (see
[0003 — Kernel model](../decisions/0003-kernel-model.md)): even the ones
currently running in kernel space for bring-up simplicity are built behind
an interface a userspace version of the same driver could use unchanged.
PCI enumeration is what discovers real hardware in the first place (the
AHCI storage controller and the e1000 network controller, in particular)
rather than assuming fixed hardware addresses.

## Networking: a real TCP/IP stack

The network stack is built in layers over the e1000 driver: Ethernet, ARP,
IPv4, UDP, TCP, a sockets API, DHCP, DNS, and IPv6 — each verified over both
loopback and emulated hardware. As with storage (see
[filesystem.md](filesystem.md)), higher layers talk to a stable sockets
interface without needing to know the specifics of the NIC underneath.

## The userspace runtime

"Runtime" here means the layer between raw syscalls and a program that
looks like normal code: startup/entry glue, a syscall wrapper layer, and
enough of a standard-library-shaped surface that userspace programs don't
have to hand-roll syscall numbers and calling conventions themselves. Both
a kernel-side runtime (for kernel-space test/utility code) and a userspace
runtime exist, sharing the same underlying syscall ABI — and every syscall
that layer makes is checked against the capability model (see
[capabilities.md](capabilities.md)) before it's allowed to touch a
resource.

## Where this shows up later

- The [ELF loader](processes.md#the-elf-loader) is what turns a compiled
  binary into a running process using this same runtime layer.
- The desktop's applications (see [desktop.md](desktop.md)) are ordinary
  userspace processes built on this runtime, not special-cased kernel code.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M12–M17, M24–M26 range).
