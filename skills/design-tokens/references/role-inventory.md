# Role inventory

The complete list of roles a production system needs, by category. Build against this list rather than adding tokens as screens demand them, or the system ends up shaped like whichever screen shipped first.

Use it two ways. On a new system, cover every role before writing a component. On an audit, mark each role `covered`, `missing` or `over-covered`, and report those three counts ahead of any individual finding.

A role marked missing is usually not absent from the product. It is hardcoded somewhere, which is worse, because nothing can theme it.

## Color

| Group | Roles |
| --- | --- |
| Surface | page background, surface, raised (menus, popovers, tooltips), sunken (inputs, wells, code blocks), overlay scrim |
| Text | primary, secondary, tertiary, disabled, inverse, on-accent, link |
| Border | subtle, default, strong, separator, focus ring |
| Accent | subtle background, subtle border, solid, solid hover, solid active, text |
| Status | per status the product actually renders: subtle background, border, solid, text |

Separator and border stay separate roles even at one value. A separator divides content and a border encloses a control, and they part ways the first time inputs are restyled.

Ship only the status ramps the product renders. A `warning` ramp nothing imports is maintenance for zero pixels.

## Space

| Group | Roles |
| --- | --- |
| Stack | `xs` through `3xl`, vertical rhythm between blocks |
| Inline | `xs` through `lg`, horizontal gaps between peers |
| Inset | `xs` through `xl`, padding inside a container |
| Layout | section gutter, content max-width, grid gap |

Stack and inset stay separate at equal values, for the same reason separator and border do. They diverge the first time cards get roomier without changing the page rhythm.

## Type

One token per text role, each a composed `font` shorthand plus a tracking token.

| Role | Renders |
| --- | --- |
| display | Hero and page-defining headlines |
| heading-1 through heading-3 | Section headings, however many levels the product has |
| body | Default reading text |
| body-sm | Dense text, table cells, secondary rows |
| label | Form labels, buttons, chips |
| caption | Metadata, timestamps, helper text |
| code | Monospace |

Do not ship a level nothing renders. Three heading levels used is a complete ramp, and six with two in use is four decisions nobody needs to make.

## Radius

| Role | Renders |
| --- | --- |
| sm | Chips, badges, inputs |
| md | Cards, panels |
| lg | Modals, sheets |
| full | Pills, avatars |

Nesting rule: a child inside a padded parent takes the parent radius minus the padding, so the curves stay concentric. Where that lands between steps, add the step rather than eyeballing it.

## Elevation

One composed shadow per level, never assembled from parts.

| Role | Renders |
| --- | --- |
| raised | Cards lifting off the page |
| overlay | Dropdowns, popovers, tooltips |
| modal | Dialogs and sheets |

Elevation and z-index are different roles. A shadow says how high something looks and a z-index says what covers what. A system that maps one to the other breaks the first time a flat element needs to sit on top.

## Motion

Duration and easing ship paired, named by intent.

| Role | Renders |
| --- | --- |
| micro | Hover, focus and other state changes under 100ms |
| enter | Something appearing |
| exit | Something leaving, shorter than enter |
| emphasis | A deliberate, noticed transition |

A reduced-motion fallback is part of the system, not a component concern. Ship the roles resolving to `0s` inside `@media (prefers-reduced-motion: reduce)` so every consumer inherits the behavior.

## Z-index

| Role | Renders |
| --- | --- |
| base | Default stacking |
| sticky | Sticky headers, bottom bars |
| overlay | Dropdowns, popovers |
| modal | Dialogs |
| toast | Notifications above everything |

Name the layer, never the number. `--z-modal` survives an insertion between layers and `--z-400` does not.

## Reporting the inventory

```
Roles covered      27 / 31
Roles missing       4   (color-text-disabled, space-inline-xs,
                         motion-micro, z-toast)
Over-covered        2   (bg-surface has 3 tokens, text-secondary has 2)
Hardcoded in situ   8   occurrences of a missing role, inline
```

Those four numbers describe a system's health better than its token count does.
