# Capabilities

> 🇧🇷 [Ler em Português](../../pt-br/architecture/capabilities.md)

Covers the security-model half of Memory/Processes/Userspace (M18–M23): how
Lynox decides whether a process is allowed to do something.

## Why not Unix-style uid/gid

The traditional Unix model checks *who you are* (a user/group id) against
*who owns the resource* — a global, ambient identity that every syscall
implicitly carries. That model has a well-known weakness: any code running
as a given user inherits *all* of that user's access, whether or not it
actually needs most of it. A bug or a malicious dependency in one part of a
process's code can reach anything the process's user is allowed to touch.

## Capability-based syscalls

Lynox instead requires a process to hold an explicit, unforgeable
capability for a specific resource before a syscall touching that resource
is allowed to succeed — there's no ambient "I'm root, so I can touch
anything" escape hatch. A capability is granted deliberately (e.g., when a
file is opened, or when a resource is explicitly handed to a child process),
and a process that was never given a capability for something cannot
reach it, regardless of what user context it's nominally running under.

This is the same family of idea referenced in the project's foundational
kernel-model decision (see
[overview.md](overview.md#kernel-model-hybrid-with-microkernel-discipline-at-the-boundaries))
as a reference point for IPC and access control design in modern
capability-oriented microkernels — adopted for the same reason: it composes
better with strong isolation between processes than an ambient-identity
model does, and it makes "this specific piece of code only got the access
it was actually handed" a property the kernel can enforce, not just a
convention.

## Where this shows up later

- Process creation (see [processes.md](processes.md)) is where capabilities
  get granted to a new process in the first place.
- The [filesystem](filesystem.md), network stack, and every other
  resource-owning subsystem check a capability before honoring a request,
  rather than trusting a caller's identity.
- [IPC](ipc.md) endpoints are themselves capability-checked resources, not
  an ambient channel any process can reach.

See [`milestones/en/milestones.md`](../../../milestones/en/milestones.md) for this phase's place in the milestone checklist (M18–M23 range).
