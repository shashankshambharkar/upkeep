---
name: design-tokens
description: Helps you design, audit and shrink a design token system. Covers tier architecture, naming grammar, the role inventory a system needs, collapsing a bloated set and keeping every token resolvable in every theme.
---

# Design tokens

A token system is a role inventory with values attached, not a list of values with names attached. Almost every bloated system got that backwards: it grew a token per screen that needed one, and ended up with hundreds of names nobody can choose between.

Size is the symptom, roles are the diagnosis. Before proposing a single deletion, count the roles the product actually renders and compare that to the token count. A system with 240 tokens serving 27 roles is not large, it is 213 aliases deep.

Color values, ramp construction and contrast measurement belong to `better-colors` where that skill is installed. This skill owns the architecture those values sit in, across every token type. Component prop and variant design belongs to `component-api`. Finding violations across a codebase belongs to `design-system-audit`.

## Exact, or say you did not check

Token work has verifiable answers. Whether a token is referenced, whether it resolves in dark mode, whether two tokens hold the same value: all of these are `grep` away. Run the check. Never report a token as unused, duplicated or missing without the command that proved it, and report anything you could not run as unverified rather than inferring it.

## Three tiers, and the third is an exception

**Primitive** names a value: `--grey-500`, `--space-4`, `--duration-fast`. It describes what the thing is, never what it is for, and no component ever reads one.

**Semantic** names a job: `--color-text-secondary`, `--space-inline-sm`, `--motion-exit`. It points at a primitive and it is the only tier components reference.

**Component** names a documented divergence: `--button-danger-bg`. Add one where a component genuinely departs from the system and you want the departure visible. One is an exception worth reading. Twenty mean the semantic tier is missing roles, and the fix is upstream.

Three tiers is a ceiling, not a pipeline. A value does not have to pass through all three.

## A component may not read a primitive

This is the rule the whole architecture rests on. When a component needs something the semantic tier cannot express, the semantic tier is missing a role: add the role rather than reaching past it.

Reaching past it is how theming dies. A codebase applying `--grey-500` directly has no seam to repoint, so a second theme means auditing every usage to work out which meant "secondary text" and which just wanted grey. See [tiers.md](references/tiers.md) for the enforcement patterns, including the lint rule that makes the violation fail CI rather than review.

## Count roles, not values

Audit a system by its role inventory, not its line count. [role-inventory.md](references/role-inventory.md) lists every role a production system needs across color, space, type, radius, elevation, motion and z-index. Walk it and mark each role as covered, missing, or over-covered.

Three roles with one token each is a complete system. One role with three tokens is a decision nobody can make.

## Name for the job, never the value or the first use

Semantic names follow one grammar and never deviate:

```
--{category}-{role}-{variant}-{state}
```

`--color-bg-surface-raised`, `--color-text-secondary`, `--space-stack-lg`, `--motion-enter-slow`. Every segment is optional except category and role, and the order never changes.

Three names to refuse on sight. `--color-blue-button` names the value, and dies the day the button turns green. `--color-sidebar-grey` names the first place it was used, and lies the second time it is used. `--color-primary` next to `--color-text-primary` means two different things by one word: reserve `accent` for the brand and let `primary` mean "most prominent of its group". More cases in [naming-grammar.md](references/naming-grammar.md).

## Collapse by role collision, not by value collision

The move that shrinks a system is not deduplication. Two tokens holding `#737373` are one token only when they also serve one role. `--color-text-tertiary` and `--color-border-strong` can share a value today and must stay separate, because they diverge the first time someone lightens borders.

So group by role first, then by value inside each role group. Every group of two or more inside one role is a collapse. Every pair that matches on value across two roles is a coincidence, and merging it is the bug that ships six months later. [audit-method.md](references/audit-method.md) is the full procedure with the commands for each pass.

## Ship type as whole roles

A text style is a role, not a set of parts. Ship `--type-body-font` as a complete `font` shorthand with a matching `--type-body-tracking`, so a component never assembles a style from loose size, weight and leading tokens.

Loose parts guarantee drift: the fourth component to want body text picks `--font-size-3` with the wrong line height, and nothing catches it. The same applies to elevation, which ships as one composed shadow per level rather than offset, blur and color separately.

## Step the space ramp, do not enumerate it

A space ramp is a function, not a list of numbers somebody needed. Start at `4px` and step by a consistent rule up to the largest gap the product actually renders.

Layout space and inset space are separate roles even at equal values. `--space-stack-md` between stacked blocks and `--space-inset-md` inside a card share `16px` until the day cards get roomier, and a system that conflated them cannot make that change.

## Pair duration with easing and name the intent

A motion token is never a bare number, because an entrance is a duration and a curve together. `--motion-enter` carries both, so no component can pair a `150ms` with an ease-out meant for `400ms`.

Name by intent, not by speed. `--motion-exit`, `--motion-emphasis`, `--motion-micro`, never `--duration-200`. Exits run shorter than entrances, so intent names encode a decision that a number does not.

## Every semantic token resolves in every theme

A semantic token that exists in `:root` and not in the dark block is a hole, and it renders as the light value on a dark surface. Declare the full semantic set in every theme, even where two themes hold the same value.

Primitives stay theme-invariant. A theme repoints semantics at different primitives, and it never redefines what `--grey-500` means. Pick one switching mechanism, either `prefers-color-scheme` or a `data-theme` attribute, and use it for every token. A system with some tokens on the media query and some on the class has two half-themes. See [theming.md](references/theming.md).

## Mirror the structure into Figma, not just the values

Figma variable collections map to tiers: one primitive collection, one semantic collection aliasing into it. Modes on the semantic collection are themes.

Syncing values alone produces a Figma library where a designer picks `grey-500`, because the semantic layer is not there to pick from. The seam that exists in code then does not exist in design. Name-match exactly, including the `--` convention stripped consistently, so a token is greppable from either side.

## Before you finish

| Mistake | Fix |
| --- | --- |
| A raw value in a component where a role token exists | Reference the semantic token |
| A raw value in a component where no role token exists | Add the semantic role, then reference it |
| A primitive referenced inside a component | Add or point a semantic token at it, and lint the tier boundary |
| Token named after its value (`--color-blue-border`) | Rename to its role (`--color-border-accent`) |
| Token named after its first use (`--color-sidebar-bg`) | Rename to the role it fills (`--color-bg-sunken`) |
| `--color-primary` for the brand and `--color-text-primary` for body text | Use `accent` for the brand, `primary` for most-prominent-of-group |
| Two tokens merged because their values matched | Split them back out; merge on role, never on value |
| A role with three tokens and no rule for choosing between them | Keep the one the product renders and delete the rest |
| Font size, weight and line height as separate tokens components assemble | Compose a whole `font` shorthand per type role |
| Shadow shipped as separate offset, blur and color tokens | Compose one shadow token per elevation level |
| One space ramp used for both gaps and padding | Split `stack` and `inset` roles even at equal values |
| A bare `--duration-200` token | Pair duration with easing and name the intent |
| A semantic token declared in `:root` only | Declare the full semantic set in every theme block |
| `prefers-color-scheme` setting some tokens and `[data-theme]` setting others | Pick one mechanism and move every token onto it |
| A primitive redefined inside a theme block | Repoint the semantic token instead; primitives are theme-invariant |
| Figma variables synced as one flat collection | Split into primitive and semantic collections, semantic aliasing into primitive |
| A token declared and never referenced | Delete it, after grepping the full consuming surface |

## Reporting

**Severity.** `HIGH` breaks theming or renders the wrong value: a tier violation, a token missing from a theme, a role collision merged by value. `MEDIUM` is a naming or structure failure that will cause drift: a value-named token, loose type parts, a duplicated role. `LOW` is an unused token or an inconsistent but harmless name.

**Verification.** Grep the full consuming surface before reporting a token unused, and name the paths you searched. Resolve every semantic token in every declared theme before reporting coverage. Report a duplicate only after checking both roles, not both values. Anything you could not run is `Not verified`.

**Format.** Group findings under the principle each violates, ordered by severity, one row per root cause with every location it appears in:

| Severity | Token | Location | Finding | Fix |
| --- | --- | --- | --- | --- |

`Location` is `path/to/file:line`. Lead the report with the role inventory count: roles covered, roles missing, tokens per role. That number is the finding everything else explains.

End with `Block` when any `HIGH` remains and `Approve` otherwise, leaving the rest in the table as work to do. With nothing to report, state "No actionable token findings" and give the role inventory count anyway.
