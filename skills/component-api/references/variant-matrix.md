# Variant matrix

Declare the axes, then the cells that must not ship. For why a flat enum is the wrong shape, see the principle **Variants are axes, not one flat list** in `SKILL.md`.

## Declaring axes

```ts
type ButtonAxes = {
  appearance: "solid" | "outline" | "ghost";
  tone: "neutral" | "accent" | "danger";
  size: "sm" | "md" | "lg";
};
```

Three axes at 3 × 3 × 3 is 27 renderable buttons from three props. The same surface as a flat enum needs 27 names, and each new appearance adds nine more.

An axis earns its place when the choice on it is independent of the others. `size` does not constrain `tone`, so they are two axes. If picking `appearance: "ghost"` forced `tone: "neutral"`, they were one axis pretending to be two.

## Give every axis a default

```ts
appearance = "solid", tone = "neutral", size = "md"
```

A component with three required axis props is a component nobody uses casually. The default is the shape the product renders most, so count call sites rather than picking the first value in the union.

## Name the invalid cells

Some cells are renderable and wrong. Declare them rather than documenting them, so the type system refuses them.

```ts
// A ghost button has no fill, so a danger tone has nothing to colour
// except the label, which reads as a link rather than a destructive action.
type ValidButton =
  | { appearance: "solid" | "outline"; tone: "neutral" | "accent" | "danger" }
  | { appearance: "ghost"; tone: "neutral" | "accent" };
```

Where the type system cannot express the constraint, or the component ships to a JS consumer, warn once at runtime in development and render the nearest valid cell. Silently rendering the invalid one teaches consumers it is supported.

Carry the reason in a comment beside the exclusion. A constraint without a reason gets deleted by whoever needs that cell next quarter.

## Every cell needs a job

An axis value that exists because the enum looked incomplete is a value nobody can choose. Before adding one, name the situation it is for in a sentence a consumer could match against.

| Value | Use when |
| --- | --- |
| `appearance: "solid"` | The primary action of a view. One per view. |
| `appearance: "outline"` | A secondary action sitting beside a solid one. |
| `appearance: "ghost"` | A tertiary action in dense UI, toolbars and table rows. |
| `tone: "danger"` | The action destroys data the user cannot recover. |
| `size: "sm"` | Inside a dense container: table row, toolbar, chip group. |

That table is the documentation and the metadata. A consumer reads it to choose, and an agent reads it to choose correctly, which is the same problem.

## Bind variant styling to component tokens, not values

```css
/* Correct: the cell resolves through the token layer. */
.button[data-appearance="solid"][data-tone="danger"] {
  background: var(--color-danger-solid);
  color: var(--color-text-on-accent);
}

/* Wrong: the cell holds a value, and theming cannot reach it. */
.button.solid.danger { background: #dc2626; }
```

Every cell resolving through semantic tokens means a theme changes 27 buttons by repointing four tokens. See `design-tokens` for the tier rule this depends on.

## When an axis should be a new component

An axis is the wrong tool when a value changes the semantics rather than the appearance. `appearance: "link"` on a Button renders an anchor, changes the keyboard behavior, changes what right-click offers and changes what a screen reader announces. That is a different component.

The test is behavior, not markup. A cell that only changes how something looks is a variant, and a cell that changes what the thing is or how it responds is a component. A `size` axis stays an axis at every value. A `variant: "icon-only"` that also removes the accessible label is a component with a different contract.
