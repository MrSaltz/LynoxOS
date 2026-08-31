# UI Guidelines

> 🇧🇷 [Ler em Português](../../pt-br/design/ui-guidelines.md)

Design principles specific to the [UI toolkit](../architecture/ui.md) and
[window system](../architecture/window-system.md), on top of the general
rules in [`coding-principles.md`](coding-principles.md).

## One shared theme, no per-widget styling

Every widget draws against a single shared `Theme` (colors, spacing)
instead of hardcoding its own visual constants. This isn't a visual
preference so much as a consistency guarantee: a new widget can't
accidentally look inconsistent with existing ones, because it never had the
option of defining its own colors in the first place.

## State is visible, not just logical

Interaction states that matter functionally (focused vs. unfocused,
hovered vs. pressed, a window that's focused vs. one that isn't) are always
given a distinct visual treatment, not tracked only internally. A user
should never have to guess which window will receive the next keystroke,
or whether a button registered a click — the state the system is actually
in and the state it visually appears to be in are kept in lock-step by
construction.

## Mechanisms are built and proven independently before being wired together

Every window-management mechanism (drag, resize, minimize/maximize, edge
snapping, modal blocking, popup dismissal) was built and verified as its
own independently testable unit before being connected to the live desktop
session (see
[`docs/en/architecture/desktop.md`](../architecture/desktop.md)). This
mirrors the general "small, isolated changes" rule in
[`coding-principles.md`](coding-principles.md#small-isolated-changes),
applied specifically to interactive behavior: a bug found later is much
easier to localize when each mechanism already has its own regression
coverage from before it was ever combined with the others.

## No hidden global escape hatches

Modal windows block interaction with everything behind them until
dismissed — there's no back door for an event to reach a window it
shouldn't currently be able to receive input on. The same discipline that
governs [capability-based security](../architecture/capabilities.md) (no
ambient access, only explicit grants) shows up here as a UI rule: a
window's ability to receive input is explicit and enforced, not a
convention other code is expected to respect.
