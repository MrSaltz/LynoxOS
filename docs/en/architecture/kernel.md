# Kernel Core

> 🇧🇷 [Ler em Português](../../pt-br/architecture/kernel.md)

Covers the Foundation phase (M0–M7): the part of the system that exists
before there's a scheduler, a filesystem, a network stack, or a screen to
draw on. For what's built directly on top of this, see
[threads.md](threads.md) (scheduling and concurrency) and
[ipc.md](ipc.md) (inter-process communication).

## Bring-up sequence

After Limine hands off control (see [overview.md](overview.md#boot-flow)),
the kernel is responsible for everything a bare-metal environment doesn't
give you for free:

1. **GDT/IDT** — the kernel builds its own Global Descriptor Table and
   Interrupt Descriptor Table rather than trusting whatever the bootloader
   left behind, since it needs full control over segment/privilege
   semantics and exception/interrupt vectors going forward.
2. **Serial console** — the earliest possible output channel (COM1), used
   before a framebuffer console exists.
3. **Interrupts** — the PIC (and later APIC-adjacent handling) is
   programmed so hardware interrupts (timer, keyboard, mouse, disk,
   network) can actually reach the kernel instead of triggering undefined
   behavior.
4. **Physical memory** — a frame allocator (bitmap-based: one bit per
   physical page frame, tracking free/used) turns the raw memory map Limine
   provided into something the rest of the kernel can request pages from.
5. **Heap** — a kernel heap allocator sits on top of the frame allocator so
   Rust's `alloc` crate (`Vec`, `Box`, `Rc`, etc.) works inside the kernel,
   which is what makes writing kernel code in idiomatic Rust practical at
   all in a `no_std` environment.

## Where this shows up later

- The frame allocator underpins [virtual memory](memory.md).
- The timer interrupt programmed here is what drives the preemptive
  scheduler (see [threads.md](threads.md)).
- The interrupt infrastructure set up here is what every driver, the
  [userspace layer](userspace.md), and the
  [live desktop session](desktop.md) receive hardware events through.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M0–M7).
