# Security Policy

## Scope

This repository contains documentation only — no kernel, driver, or
userspace source code, and no compiled binaries or images are published
here. There is no live, deployed Lynox service reachable over a network,
and nothing in this repository is installed or executed by a reader. That
significantly narrows what a "security report" against this repository can
mean:

- **In scope**: a design-level security concern about something described
  in [`docs/en/architecture/`](../docs/en/architecture/overview.md) or
  [`docs/en/decisions/`](../docs/en/decisions/) — for example, a real flaw
  in the reasoning behind the capability model, or an isolation guarantee
  the docs claim that seems logically unsound as described.
- **Out of scope**: vulnerabilities in the actual kernel implementation —
  there's no source published here to audit. If/when kernel source is
  published, this policy will be updated with a real reporting process for
  implementation-level findings.

## Reporting a concern

For a design-level concern as described above, open an Issue using the
"Architecture question / discussion" template
([`.github/ISSUE_TEMPLATE/architecture.yml`](ISSUE_TEMPLATE/architecture.yml)).
If you believe a concern shouldn't be discussed in a public Issue before
the maintainer has a chance to review it, use GitHub's private vulnerability
reporting for this repository (under the repository's Security tab) instead
of posting details publicly.

## What to expect

Because this is documentation, not running code, there's no patch/release
cycle to speak of — a valid concern results in the relevant document being
corrected or clarified, with credit to the reporter if they'd like it.
