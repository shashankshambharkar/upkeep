# Theming

A theme repoints the semantic tier and touches nothing else. Everything below follows from that: if a theme is editing primitives or components are reading them, the system does not have a theming seam, it has a find-and-replace.

## Pick one switching mechanism

Two mechanisms in one system produce two half-themes, where some tokens follow the OS and some follow the toggle, and no combination renders correctly.

**`[data-theme]` on the root element** is the better default. It supports an explicit user choice, which `prefers-color-scheme` alone cannot, and it degrades to the system preference with one extra block:

```css
:root { /* light semantics */ }

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { /* dark semantics */ }
}

:root[data-theme="dark"] { /* dark semantics */ }
```

The `:not([data-theme="light"])` guard is what makes an explicit light choice win over a dark OS setting. Without it the toggle works one way only.

The dark semantics appear twice, so define them once in a shared block and reference it, or accept the duplication and check both on every edit. What is not acceptable is defining half in one and half in the other.

## Declare the full semantic set in every theme

A token declared in `:root` and absent from the dark block does not fall back to something sensible. It renders the light value on a dark surface, usually as invisible text, and usually on a screen nobody opened during review.

Check for holes rather than trusting the file:

```bash
rg -oP -- '--color-[a-z0-9-]+(?=\s*:)' src/styles/tokens.css \
  | sort -u > /tmp/all-tokens.txt

awk '/\[data-theme="dark"\]/,/^}/' src/styles/tokens.css \
  | rg -oP -- '--color-[a-z0-9-]+(?=\s*:)' | sort -u > /tmp/dark-tokens.txt

comm -23 /tmp/all-tokens.txt /tmp/dark-tokens.txt
```

Anything printed is a hole. Declare it in the dark block, even where the value is identical, because an identical value stated on purpose is a decision and an absent one is an accident.

## Primitives stay theme-invariant

```css
/* Correct: the semantic moves, the primitive means one thing always. */
:root                  { --color-text-secondary: var(--grey-500); }
[data-theme="dark"]    { --color-text-secondary: var(--grey-600); }

/* Wrong: --grey-500 now means two different colors. */
[data-theme="dark"]    { --grey-500: #a3a3a3; }
```

The wrong version works until something references `--grey-500` expecting the ramp, and then the ramp is not a ramp. It also makes the Figma mirror impossible, because a primitive collection with modes is not a primitive collection.

## Dark is not the light palette reversed

Reversing the ramp is the starting point, not the result. Three adjustments always follow.

- **Reduce vividness.** A saturated accent that reads correctly on white glares on near-black. Drop chroma and recheck.
- **Widen the dark end.** Steps that separated cleanly at the light end collapse at the dark end, so `bg` and `surface` stop reading as two planes. Spread them until they do.
- **Re-measure every pair.** Contrast does not survive the flip. Measuring belongs to `better-colors` where that skill is installed.

Elevation inverts too. Light themes lift with shadow and dark themes lift with lightness, so a dark theme whose raised surface is only a shadow reads flat. Give `--color-bg-raised` a lighter value in dark and keep the shadow subtle.

## More than two themes

A third theme, a white-label brand or an increased-contrast variant all work the same way, and all of them break on the same thing: a token whose name encodes an assumption only one theme holds.

`--color-bg-page` survives any theme. `--color-bg-white` survives exactly one. Audit names for baked-in assumptions before adding the second theme, not during.

Where a brand theme needs a hue the primitive ramps do not have, add a primitive ramp rather than putting a raw value in the semantic tier. The semantic tier aliases and never holds values.

## Reduced motion belongs here

Motion roles resolve to `0s` under the preference, once, at the token layer:

```css
@media (prefers-reduced-motion: reduce) {
  :root {
    --motion-micro: 0s linear;
    --motion-enter: 0s linear;
    --motion-exit: 0s linear;
    --motion-emphasis: 0s linear;
  }
}
```

Every consumer then inherits the behavior without knowing about it. A system that leaves this to components has as many implementations as it has animated components, and the newest one always forgot.
