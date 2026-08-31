# Contributing to Lynox

## Current stance

Lynox's kernel, driver, and userspace source code is developed in a private
repository while the project is still in active, early-stage development.
**This means code contributions (pull requests) to the kernel aren't being
accepted right now** — there's no public source to open a PR against, and
that's intentional for this stage of the project (see
[`README.md`](../README.md#public-vs-private) for why).

This isn't a permanent stance. If/when the source is published, this
document will be updated to reflect what contributions look like at that
point.

## What Issues are for, right now

Issues on this repository are open for:

- **Documentation problems** — a broken link, a claim that contradicts the
  changelog, a screenshot/demo that doesn't match its description. Use the
  "Documentation / demo issue" template.
- **Roadmap suggestions and questions** — something you think should be on
  the roadmap, or a question about a milestone's scope. Use the "Roadmap
  suggestion" template. Note: Lynox's own rule is that no feature gets added
  "because it'd be cool" — every milestone needs a concrete consumer or
  architectural need — so a suggestion is a starting point for discussion,
  not a guarantee it gets scheduled.
- **Architecture questions and discussion** — asking why a design decision
  was made the way it was, or pointing out something that seems
  inconsistent between two documents. Use the "Architecture question /
  discussion" template.

Issues that ask for the private source code, or that report a "bug" in
behavior that can only be observed by having the source, will be closed with
an explanation rather than acted on — there's nothing actionable to do with
them right now.

## Tone

This project's engineering discipline (see
[`docs/en/design/coding-principles.md`](../docs/en/design/coding-principles.md))
values honest, specific feedback over vague praise or vague criticism — if
something in the docs seems wrong, say specifically what and why; that's
genuinely useful here.
