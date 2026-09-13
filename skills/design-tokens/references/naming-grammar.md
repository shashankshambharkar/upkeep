# Naming grammar

One shape for semantic tokens, and it never varies:

```
--{category}-{role}-{variant}-{state}
```

Category and role are required. Variant and state are added only when a system genuinely has more than one, and the order never changes.

```css
--color-bg-surface
--color-bg-surface-raised
--color-text-secondary
--color-border-accent
--color-accent-solid-hover
--space-inset-md
--space-stack-lg
--type-body-font
--radius-md
--elevation-overlay
--motion-enter
--z-modal
```

Primitives use a different and simpler shape, because they name a value rather than a job: `--{family}-{step}`, as in `--grey-500`, `--blue-600`, `--space-4`, `--duration-fast`.

## Names to refuse on sight

| Name | Problem | Fix |
| --- | --- | --- |
| `--color-blue-button` | Names the value. Dies the day the button turns green. | `--color-accent-solid` |
| `--color-sidebar-grey` | Names the first use. Lies at the second. | `--color-bg-sunken` |
| `--color-primary` beside `--color-text-primary` | One word, two meanings. | `--color-accent-solid` and `--color-text-primary` |
| `--grey-medium` | Word ramp. No room to insert a step. | `--grey-500` |
| `--duration-200` | Number, not intent. Pairs badly with any easing. | `--motion-enter` |
| `--z-400` | Number, not layer. Breaks on insertion. | `--z-overlay` |
| `--color-text-light` | Ambiguous under theming. Light text or light-theme text? | `--color-text-secondary` |
| `--spacing-card-top` | Component concern wearing a system name. | `--space-inset-md`, applied by the card |
| `--color-error` and `--color-danger` in one system | Two vocabularies for one status. | Pick one and grep the other to zero |

## Pick one word per concept and grep the rest to zero

A system that says `bg` in some places and `background` in others has one concept and two vocabularies, and every consumer has to guess. Fix the vocabulary before fixing anything else, because every later rename compounds it.

| Concept | Word | Not |
| --- | --- | --- |
| Background | `bg` | `background`, `fill`, `surface-color` |
| Foreground text | `text` | `fg`, `foreground`, `content`, `ink` |
| Brand color | `accent` | `primary`, `brand`, `theme` |
| Most prominent of a group | `primary` | `main`, `default`, `strong` |
| Least prominent visible | `subtle` | `muted`, `faint`, `weak`, `light` |
| Destructive status | `danger` | `error`, `negative`, `destructive` |
| Interior padding | `inset` | `padding`, `pad` |
| Vertical gap | `stack` | `gap-y`, `vertical`, `block` |

The right column is not wrong in general. It is wrong alongside the left column in one system. `content` and `ink` are good words, and a system that has settled on either should grep `text` to zero instead.

## Ordering states

Where a role carries states, use `default`, `hover`, `active`, `disabled`, in that order, and omit `default` from the name:

```css
--color-accent-solid
--color-accent-solid-hover
--color-accent-solid-active
--color-accent-solid-disabled
```

`--color-accent-solid-default` reads as a fifth state that is different from the base, and someone will eventually wire it that way.

## Renaming safely

A rename is three steps, and skipping the middle one breaks consumers you do not own.

1. Add the new name pointing at the same value.
2. Alias the old name to the new one and mark it deprecated in a comment with a removal date.
3. Grep the old name to zero across every consuming surface, then delete it.

```bash
rg --no-heading --line-number 'color-sidebar-grey' src/ docs/ .storybook/
```

Deleting on the same commit as the rename is only safe when you can grep every consumer. A published library cannot, so step 2 is not optional there.
