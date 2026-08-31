# Filesystem

> 🇧🇷 [Ler em Português](../../pt-br/architecture/filesystem.md)

Covers the storage half of the Hardware/Drivers/Networking phase (M12–M17):
turning a raw storage controller into a real filesystem every other
subsystem can read and write through.

## AHCI: a real storage driver

The storage driver talks to a real AHCI controller — DMA transfers, command
lists, FIS (Frame Information Structures), and PRDT (Physical Region
Descriptor Tables) — rather than a simplified or emulated block interface.
Discovering the controller in the first place goes through PCI enumeration
(see [userspace.md](userspace.md#driver-architecture)), not a fixed,
assumed hardware address.

## The Virtual Filesystem (VFS)

On top of the raw block driver sits a Virtual Filesystem layer with its own
persistent filesystem implementation and an LRU write-through cache. Every
higher layer — a process's open file descriptors, the desktop's File
Manager (see [desktop.md](desktop.md)) — talks to a single, consistent
filesystem interface regardless of what's actually backing a given path,
the same way the userspace runtime gives programs a stable syscall surface
regardless of which driver ends up handling a given request (see
[userspace.md](userspace.md#the-userspace-runtime)).

## Where this shows up later

- The image/asset pipeline (see
  [graphics.md](graphics.md#the-image--asset-system-m35-in-progress)) loads
  its raw bytes through this same VFS layer, not a separate, ad-hoc I/O
  path — a loaded image and a text file opened by the File Manager go
  through the exact same read mechanism.
- Every capability-checked file access (see
  [capabilities.md](capabilities.md)) is enforced at this layer.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M12–M17 range).
