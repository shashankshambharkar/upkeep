# Detection

One command per pass. Every one assumes `rg` and takes `SRC` as the consuming surface and `SYS` as the system source, set once:

```bash
SRC=(src/app src/features)      # what should be using the system
SYS="src/design-system"         # the system itself, excluded from SRC
```

`SRC` is an array, and every use of it below is `"${SRC[@]}"`. This is not style. A plain
string expanded as `$SRC` word-splits in bash and does not in zsh, which is the default shell
on macOS. There it becomes one path named `src/app src/features`, every pass returns zero and
the report reads as a clean codebase.

Adjust the globs to the project's stack before running. A command that returns zero because it grepped the wrong extension is the one failure mode that looks like success. Sanity-check each pass against a value you already know is there.

Four portability traps, all of which fail quietly rather than loudly:

- **A multi-path variable must be an array.** zsh does not word-split `$SRC`, so a string of two paths becomes one path that does not exist. Use `SRC=(a b)` and `"${SRC[@]}"`, which behave the same in both shells.
- **Every `rg` flag goes before `--`.** Token names start with `--`, so the separator is required, and anything after it is a path. `rg -o -- 'pattern' -P src/` silently treats `-P` as a filename and the lookahead becomes a parse error.
- **A literal search for a token name needs `-F --`.** `rg -F "--grey-500"` reads the name as a flag. `rg -F -- "--grey-500"` does not.
- **BSD `sed` on macOS has no `\s` or `\d`.** Use `[[:space:]]` and `[[:digit:]]`, which work on both platforms.

## Inventory: declared tokens

CSS custom properties:

```bash
rg --no-heading -oP -g '*.css' -- '--[a-z0-9-]+(?=\s*:)' "$SYS" \
  | sed 's/^[^:]*://' | sort -u > tokens.txt
wc -l tokens.txt
```

A JS or TS token object, via the module itself rather than a regex:

```bash
node -e '
  const t = require("./src/design-system/tokens.js");
  const walk = (o, p = []) =>
    Object.entries(o).flatMap(([k, v]) =>
      v && typeof v === "object" ? walk(v, [...p, k]) : [[...p, k].join(".") + "\t" + v]
    );
  console.log(walk(t).join("\n"));
' > tokens.tsv
```

Split the list by tier. Anything matching the primitive ramp shape is tier 1, and everything else is tier 2.

## Inventory: exported components

Read the entry point, never the source tree. A component in `src/design-system/Foo.tsx` that the barrel does not export is not adoption surface.

```bash
rg --no-heading -o 'export \{ ([^}]+) \}' -r '$1' "$SYS/index.ts" \
  | tr ',' '\n' | sed 's/ as .*//' | tr -d ' ' | sort -u > components.txt
```

## Pass 1: hardcoded values

```bash
rg --no-heading --line-number \
  '#[0-9a-fA-F]{3,8}\b|\brgba?\([^)]*\)|\bhsla?\([^)]*\)|\boklch\([^)]*\)' \
  "${SRC[@]}" -g '!*.test.*' -g '!*.stories.*' -g '!*.svg' \
  > findings-color.txt

rg --no-heading --line-number \
  '(?:padding|margin|gap|border-radius|top|left|right|bottom)\s*:\s*\d+px' \
  "${SRC[@]}" -g '!*.test.*' \
  > findings-space.txt
```

Tailwind projects need the arbitrary-value form instead, which is where hardcoding hides:

```bash
rg --no-heading --line-number '\[(#[0-9a-fA-F]{3,8}|\d+px|rgba?\([^]]*\))\]' "${SRC[@]}"
```

Three exclusions before counting. `0px` and `1px` are frequently correct and rarely tokenised. Values inside an SVG `fill` are artwork. And a color inside the system's own primitive declarations is the ramp, not a bypass.

## Pass 2: tier violations

A primitive referenced anywhere outside the token file. Match the project's primitive families rather than this list:

```bash
rg --no-heading --line-number \
  'var\(--(grey|gray|blue|red|green|amber|purple|space|duration|ease)-[0-9a-z]+\)' \
  "${SRC[@]}" "$SYS" -g '!**/tokens.css'
```

Every hit is a component reaching past the semantic tier. Report the count and the distinct tokens separately: fifty hits on one token is one missing role, and fifty hits on fifty tokens is no semantic tier at all.

## Pass 3: missing roles

Not a grep, a join. Take the distinct hardcoded values from pass 1, map each to the role it fills at its call site, then check that role against the `design-tokens` role inventory.

```bash
rg --no-heading -o '#[0-9a-fA-F]{3,8}' "${SRC[@]}" | sort | uniq -c | sort -rn | head -30
```

Work the frequent ones first. A color appearing 40 times is a role the system does not cover, and the 1-occurrence tail is mostly genuine one-offs.

For each, check whether a token already holds that value:

```bash
grep -i "$VALUE" tokens.txt
```

A match means bypass and the owner is the consuming team. No match means gap and the owner is the system team. That single check is what separates the two causes, and skipping it is how an audit blames the wrong people.

## Pass 4: dead tokens

```bash
while read -r name; do
  n=$(rg --count-matches --no-filename -F -- "$name" "${SRC[@]}" "$SYS" 2>/dev/null | paste -sd+ - | bc)
  [ "${n:-0}" -le 1 ] && echo "DEAD $name"
done < tokens.txt
```

Two false positives to clear before reporting. A name built by template (`var(--color-text-${tone})`) never appears literally, so grep the prefix. And a token unused on web may be the only thing a native export reads, so check the export targets.

## Pass 5: theme holes

```bash
rg -oP -- '--color-[a-z0-9-]+(?=\s*:)' "$SYS/tokens.css" | sort -u > all.txt
awk '/\[data-theme="dark"\]/,/^}/' "$SYS/tokens.css" \
  | rg -oP -- '--color-[a-z0-9-]+(?=\s*:)' | sort -u > dark.txt
comm -23 all.txt dark.txt
```

Anything printed renders its light value on a dark surface. Subtract the primitives first, since those are theme-invariant by design and will otherwise fill the output.

## Pass 6: shadow components

Local implementations of a pattern the system already exports. Search for the system's component names as local definitions:

```bash
while read -r c; do
  rg --no-heading --line-number "(function|const) $c\b|class $c\b" "${SRC[@]}" \
    | rg -v "$SYS" | sed "s/^/SHADOW $c: /"
done < components.txt
```

Then catch the ones under different names, by shape rather than by name:

```bash
rg --no-heading --line-number -c '<button[^>]*className' "${SRC[@]}" | sort -t: -k2 -rn | head -20
```

A file with many raw styled `button` elements is a shadow Button whatever it is called. The same heuristic works for raw `input`, `dialog` and `table`.

Classify carefully. A shadow component for a pattern the system does not export is a gap, and one duplicating an exported component is a bypass.

## Pass 7: single-consumer props

For each exported component, count the distinct files setting each prop:

```bash
COMPONENT="Button"
rg --no-heading -oUP --multiline -r '$1' -- "<$COMPONENT[^>]*?\b([a-zA-Z]+)=" "${SRC[@]}" \
  | sed 's/^[^:]*://' | sort | uniq -c | sort -n | awk '$1 <= 1'
```

Each result is a prop exactly one call site sets, which `component-api` treats as a fork. Check the call site before reporting: a prop set once because the component is used twice is not the same finding as a prop set once out of two hundred uses.

## Pass 8: bypassed imports

```bash
rg --no-heading --line-number -P -- "from ['\"].*design-system/(?!index)" "${SRC[@]}"
```

Deep imports skip the entry point, so they reach internals that were never API and they survive no refactor. Count them separately from other bypasses, because the remediation is a single find-and-replace rather than a redesign.

## Sanity check before reporting

Run one known-positive per pass. Pick a file you have read, confirm the pass finds what you saw there, and only then trust the totals. A pass returning zero is either a clean codebase or a wrong glob, and the report cannot tell the difference.
