# Audit method

The procedure for taking an existing token set apart and putting back only what earns its place. It runs in five passes, in order, and each pass only deletes what the pass before it proved deletable. Running them out of order merges tokens that should have stayed separate.

Work on a branch. Every pass ends in a commit, so a wrong merge is one revert rather than an archaeology session.

## Pass 0: inventory

Extract every declared token with its value and its declaring line. For a CSS custom property set:

```bash
rg --no-heading --line-number '^[[:space:]]*--[a-z0-9-]+[[:space:]]*:' src/ \
  | sed -E 's/^([^:]+):([0-9]+):[[:space:]]*(--[a-z0-9-]+)[[:space:]]*:[[:space:]]*(.+);.*$/\3\t\4\t\1:\2/' \
  > tokens.tsv
wc -l tokens.tsv
```

For a JS or JSON token source, walk the object and emit the same three columns. The shape you want is `name`, `value`, `location`, one row per declaration.

That line count is the number you are going to move. Write it down.

## Pass 1: delete the unreferenced

A token nothing reads is not a design decision, it is a leftover. Check each name against the full consuming surface, which includes app code, other token files, stories, docs and any Figma export script.

```bash
while IFS=$'\t' read -r name value loc; do
  n=$(rg --count-matches --no-filename -F -- "$name" src/ docs/ 2>/dev/null | paste -sd+ - | bc)
  [ "${n:-0}" -le 1 ] && echo "UNUSED $name  $loc"
done < tokens.tsv
```

The `-le 1` accounts for the declaration itself matching. Two traps before deleting anything:

- **Constructed names.** A codebase doing `var(--color-text-${tone})` never literally contains `--color-text-secondary`. Grep for the template prefix before trusting a zero.
- **Cross-platform consumers.** A token unused in web may be the only thing an iOS or Android export reads. Check the export targets, not just the app.

Commit. Re-count.

## Pass 2: group by role, then collapse inside each group

This is the pass that does the work, and the order inside it is the whole trick.

Assign every surviving token a role from [role-inventory.md](role-inventory.md), by what it is used for at its call sites rather than by its name. A token named `--color-sidebar-bg` referenced only as a card background has the role `bg-surface`, whatever it is called.

Then, inside one role group only, collapse tokens that hold the same rendered value:

```bash
sort -k2 tokens.tsv | awk -F'\t' '{ print $2 }' | uniq -d
```

Use that list as a candidate set, never as the answer. For each duplicated value, check whether every token holding it carries the same role. Same value and same role is a collapse. Same value and different roles stays split, and the note explaining why goes in the token file.

The pair to watch for is a text color and a border color that agree today. They are the most common false merge, and they diverge the first time borders are lightened.

Pick the survivor by role fit, not by usage count. The winning name is the one that describes the role, so a role with three tokens often keeps none of the three names and gets a new one.

Commit. Re-count. This is usually where a set loses most of its bulk.

## Pass 3: fill the holes the collapse exposed

Collapsing exposes roles with no token at all, because the screens needing them had been hardcoding instead. Walk [role-inventory.md](role-inventory.md) against the collapsed set and list every uncovered role.

Find what those screens use today:

```bash
rg --no-heading --line-number \
  '(#[0-9a-fA-F]{3,8}\b|rgba?\([^)]*\)|\b\d+px\b)' src/components/ \
  --glob '!**/*.test.*'
```

Every hit is either a token that should exist or a deliberate exception. Add the missing semantic roles, point them at primitives, then replace the hardcoded values. The count goes up in this pass, and that is correct.

Commit.

## Pass 4: enforce the boundary

A shrunk system regrows unless the tier rule is machine-checked. Add the lint rule from [tiers.md](tiers.md) so a component reading a primitive fails CI rather than review.

Then re-resolve every semantic token in every theme and confirm no holes, per [theming.md](theming.md).

Commit.

## Report the shape, not just the number

The headline is roles, not tokens. "240 to 27" means nothing without "covering 27 roles, up from 19 covered and 8 hardcoded". Report:

| Metric | Before | After |
| --- | --- | --- |
| Tokens declared | | |
| Roles covered | | |
| Roles with more than one token | | |
| Hardcoded values in components | | |
| Semantic tokens missing from a theme | | |

A system that went from 240 tokens to 27 while leaving 8 roles hardcoded got smaller and worse. The last two rows are what make the first row a win.
