# Tiers

Three tiers, and only the middle one is load-bearing for components. For why the boundary matters, see the principle **A component may not read a primitive** in `SKILL.md`.

## The shape

```css
:root {
  /* Tier 1 · primitive. Names a value. No component reads these. */
  --grey-200: #e5e5e5;
  --grey-500: #737373;
  --blue-500: #3b82f6;
  --space-2: 8px;
  --space-4: 16px;
  --duration-fast: 150ms;
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);

  /* Tier 2 · semantic. Names a job. The only tier components reference. */
  --color-text-secondary: var(--grey-500);
  --color-border: var(--grey-200);
  --color-accent-solid: var(--blue-500);
  --space-inset-md: var(--space-4);
  --space-stack-sm: var(--space-2);
  --motion-enter: var(--duration-fast) var(--ease-out);
}

[data-theme="dark"] {
  /* Only semantics move. Primitives mean the same thing in every theme. */
  --color-text-secondary: var(--grey-600);
  --color-border: var(--grey-300);
}
```

## Tier 3 is a note, not a layer

```css
/* Deliberate divergence: destructive actions run one step darker than the
   status ramp, because the button sits on a surface the alerts never do. */
--button-danger-bg: var(--red-600);
```

A component token is worth adding when the divergence is real and you want it visible in one place. It is not worth adding as a routine indirection. A file where every component has its own tier-3 block has three tiers of aliasing and one tier of meaning.

Count them during an audit. More than a handful means the semantic tier is missing roles, and the fix is to add those roles rather than to keep aliasing around them.

## Enforce the boundary in CI

The rule survives review for about two sprints. Make it fail the build instead.

**Stylelint**, for CSS custom properties. Allow primitives only in the token file:

```json
{
  "overrides": [
    {
      "files": ["src/**/*.css", "!src/styles/tokens.css"],
      "rules": {
        "declaration-property-value-disallowed-list": {
          "/.*/": ["/var\\(--(grey|blue|red|green|amber)-\\d+\\)/"]
        }
      }
    }
  ]
}
```

**ESLint**, for a JS or TS token object, using `no-restricted-imports` to block the primitive export outside the token module:

```json
{
  "rules": {
    "no-restricted-imports": ["error", {
      "paths": [{
        "name": "@/styles/tokens",
        "importNames": ["primitives"],
        "message": "Components read semantic tokens. Add a semantic role instead."
      }]
    }]
  }
}
```

**Grep, as a pre-commit hook**, where neither linter is set up:

```bash
rg -l 'var\(--(grey|blue|red|green|amber)-[0-9]' src/components/ && {
  echo "Primitive token referenced in a component. Add a semantic role."
  exit 1
}
```

Whichever you pick, the message matters as much as the rule. "Add a semantic role instead" tells the next person what to do. "Disallowed value" sends them to disable the rule.

## Naming the primitive ramp

Primitives are named by what they are, on a scale that has room to grow. `--grey-500`, not `--grey-medium`. A numbered ramp takes an insertion between `500` and `600` without renaming anything, and a word ramp does not.

Steps of `100` on a `50` to `950` scale is the most portable convention, because it matches what Tailwind and most exported Figma ramps already use. A `1` to `12` scale in the Radix convention works equally well. What does not work is mixing them in one system.
