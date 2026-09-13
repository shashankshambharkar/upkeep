---
name: component-api
description: Helps you design and review the public API of design system components. Covers variant axes, prop naming, when configuration should become composition, controlled state, escape hatches and deprecating a prop without breaking consumers.
---

# Component API

A component's API is the part you cannot change later. Its internals get rewritten every year and nobody notices, while a badly named prop outlives three redesigns because every consumer is holding it.

Press hard on anything in the public signature and lightly on everything behind it. A prop name, a variant axis and a required prop are permanent decisions that deserve an argument. Implementation you disagree with is a preference, and saying so costs the review its credibility.

Token architecture and naming belong to `design-tokens`. Finding API violations across a codebase belongs to `design-system-audit`. Visual and interaction craft belong to `better-ui` and `better-accessibility` where those skills are installed.

## Variants are axes, not one flat list

Declare the axes a component varies on and let a consumer pick a cell: `appearance` crossed with `size` crossed with `density`. A single `variant` prop holding `primary-large` and `secondary-small` has hidden two axes inside one enum, and every new combination multiplies the list.

Name the cells that must not ship rather than leaving them renderable. An `appearance: "ghost"` crossed with `elevation: "raised"` that makes no sense is an invalid combination, declared and type-excluded. See [variant-matrix.md](references/variant-matrix.md).

## A boolean that excludes another boolean is an enum

`isPrimary` and `isSecondary` on one component can both be true, and the component has to pick. That branch is the API admitting the props were an enum wearing two names.

The test: if setting prop B to true makes prop A meaningless, they are one prop. Replace them with `appearance="primary" | "secondary"` and give it a default. Booleans are correct for genuinely independent switches like `disabled`, `loading` and `fullWidth`.

## Configuration until the tree shape changes, composition after

A prop that sets a value is configuration and belongs on the component. A prop that changes what elements exist is composition and belongs in the children.

`icon="search"` is configuration. `renderHeader={() => ...}` is composition wearing a prop's clothes, and it should be a slot. The moment a prop's presence adds or removes a node, move it. [composition.md](references/composition.md) covers slots, compound components and the `asChild` pattern.

Three render props on one component means the component is a layout, and its consumers want the pieces.

## A prop only one consumer sets is a fork

Every prop added for one screen is a permanent branch in a shared component, paid for by everyone reading it forever. Three of them and nobody can predict what the component renders.

Refuse it, and route to one of three fixes: the need is general, so make it a variant with a name that describes the intent rather than the caller. The need is local, so compose it at the call site. The need is genuinely divergent, so fork the component on purpose and name the fork.

"We can generalise it later" never happens, because the prop already works.

## Name props for what they mean, not what they do to CSS

`tone`, not `color`. `density`, not `compact`. `emphasis`, not `bold`. A prop named after its current styling is wrong the first time the styling changes, and it leaks the implementation into every call site.

Prop names are a vocabulary, so pick one word per concept across the whole system. A library with `size` on Button, `scale` on Icon and `dimension` on Avatar has one concept and three words. [prop-naming.md](references/prop-naming.md) has the vocabulary table and the handler and boolean conventions.

## Every stateful component is controlled and uncontrolled

Ship `value` with `onChange` for the controlled path and `defaultValue` for the uncontrolled one, and let the presence of `value` decide which is active. A component that only supports one forces every consumer into a wrapper.

Warn once when a component switches between them at runtime, because that is a consumer bug that presents as the component losing state.

## Required props are the contract, so keep them unavoidable

A required prop is one a component cannot render without. Everything else takes a default. Each required prop is a thing every call site must know, and a component with five of them is a function with five arguments.

An accessible name is required when the component has no visible text, and that one is worth the friction. See `better-accessibility` for what needs a name.

## One escape hatch, and it is `className`

Accept `className` and merge it, so a consumer can solve a problem you did not predict without forking. Forward the ref and spread the remaining props onto the root element so native attributes and data attributes pass through.

Stop there. A `styles` object keyed by internal part names publishes your DOM structure as API, and the next refactor breaks every consumer. Add per-part class props only for parts a consumer demonstrably cannot reach, one at a time, each with a reason.

## Margin belongs to the parent

A component sets its own padding and never its own outer margin, because spacing between two things is a fact about their relationship and the parent owns it. A `marginBottom` prop makes every consumer responsible for a layout decision that belongs to the layout.

The same holds for width. A component fills its container or takes an explicit `fullWidth`, and the container decides.

## Deprecate in three steps, never in one

A prop leaves over three releases: warn at runtime and mark `@deprecated` so editors strike it through, then make it a type error while it still works, then remove it in a major.

Every step names the replacement in the message. "`variant` is deprecated" sends a consumer to the changelog, and "`variant` is deprecated, use `appearance`" does not. [deprecation.md](references/deprecation.md) has the codemod and the messages.

## Before you finish

| Mistake | Fix |
| --- | --- |
| Two booleans where setting one makes the other meaningless | Collapse into one enum prop with a default |
| `variant="primary-large"` | Split into `appearance` and `size` axes |
| An axis combination that renders but should not | Declare it invalid and exclude it in the type |
| A render prop or `renderX` function prop | Make it a slot in children |
| A prop only one call site sets | Generalise it, compose it, or fork the component on purpose |
| Prop named after its CSS (`color`, `bold`, `compact`) | Name the meaning: `tone`, `emphasis`, `density` |
| `size` on one component and `scale` on another | One word per concept across the library |
| Handler named `onSubmitClick` | `onSubmit` for intent, `onClick` for the raw event, never both in one name |
| Boolean named `hidden` defaulting to `true` | Name booleans so `false` is the default and the absence reads correctly |
| `value` with no `defaultValue` | Support controlled and uncontrolled, switching on `value` presence |
| A required prop that could have a sensible default | Default it; keep required props to what cannot be inferred |
| No `className` on the root | Accept and merge it, forward the ref, spread the rest |
| A `styles` or `classNames` object keyed by internal parts | Remove it; add single-part props only where a consumer cannot reach |
| `margin`, `marginTop` or `mb` prop | Delete it; the parent owns space between things |
| A prop removed in a minor release | Warn, then type-error, then remove in a major |
| A deprecation message that does not name the replacement | Name it in the message |

## Reporting

**Severity.** `HIGH` breaks consumers or cannot be fixed later without a major: a removed prop, a published internal structure, a name that will be wrong after the next redesign. `MEDIUM` is a shape problem that will cause forks: hidden axes, a single-consumer prop, a render prop. `LOW` is vocabulary drift and missing defaults.

**Verification.** Read the type signature and the implementation together, since a prop can be typed and ignored. Grep every call site before reporting a prop as single-consumer, and name the paths searched. Check the exported entry point rather than the source file, because an internal prop is not API. Report anything you could not run as `Not verified`.

**Format.** Group findings under the principle each violates, ordered by severity, one row per prop:

| Severity | Component | Prop | Finding | Fix | Breaking |
| --- | --- | --- | --- | --- | --- |

`Breaking` is yes or no, and a yes carries the deprecation path rather than the removal.

End with `Block` when any `HIGH` remains and `Approve` otherwise. With nothing to report, state "No actionable API findings" and name the components and entry points you read.
