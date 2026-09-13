# Report format

One summary block, one findings table, three ranked remediations. Nothing else, and in that order, because a system owner reads the first block in a meeting and the table afterwards.

## Summary block

```
Design system audit · web-app · 2026-09-14
Baseline: 2026-06-12

Consuming surface   src/app, src/features        1,284 files
Excluded            src/design-system, *.test.*, *.stories.*, src/legacy/*
System source       src/design-system             41 tokens, 18 components

Token adoption      87%   (2,140 tokenised / 2,461 total)      +6 pts
Component adoption  71%   (128 system / 180 instances)         -2 pts

Gaps                 4    roles the system does not cover
Bypasses           317    uses of a value the system does cover
Exceptions           2    confirmed, excluded from both scores
Not verified         1    native token export, no access
```

Five rules for this block.

- **Show both terms of every score.** A percentage with no denominator cannot be checked and cannot be compared next quarter.
- **Name the excluded paths.** An audit is only as honest as its exclusions, and the reader needs to see them without asking.
- **Report the two scores separately.** Token and component adoption move independently, and a blended number hides which one moved.
- **Carry the delta only against a real baseline.** A first audit has no arrow, and inventing one is worse than leaving it out.
- **List `Not verified` in the block, never in a footnote.** A dropped check that only appears at the bottom reads as a clean result.

## Findings table

Gaps first regardless of count, then bypasses by count descending:

| Cause | Finding | Count | Locations | Owner |
| --- | --- | --- | --- | --- |
| Gap | No token for disabled text; `#9ca3af` inline | 41 | `Input.tsx:88`, `Select.tsx:52`, `Checkbox.tsx:31` +38 | `design-tokens` |
| Gap | Shadow `Toast` in `features/notify`, no system equivalent | 1 | `features/notify/Toast.tsx:1` | `component-api` |
| Bypass | `#141414` inline where `--color-bg-surface` exists | 96 | `Card.tsx:12`, `Panel.tsx:9`, `Sheet.tsx:24` +93 | `design-tokens` |
| Bypass | Primitive `--grey-500` read inside components | 63 | `Table.tsx:44`, `Meta.tsx:17`, `Row.tsx:8` +60 | `design-tokens` |
| Bypass | Deep imports past `design-system/index` | 28 | `Header.tsx:3`, `Nav.tsx:5`, `Footer.tsx:2` +25 | `component-api` |
| Bypass | `Button` prop `elevated`, set by one call site | 1 | `Checkout.tsx:140` | `component-api` |

`Locations` gives three examples and a remainder count. A table that inlines 96 paths is a log file.

One row per root cause, never per occurrence. Ninety-six instances of one missing token reference is one decision to make, and ninety-six rows is a table that hides it.

`Owner` names the skill whose rules diagnose the fix. This skill decides no token architecture and no component API.

## Ranked remediations

Three, by score movement rather than by count, each with the projected result:

```
1. Add --color-text-disabled and replace 41 inline values
   Gap · closes 1 role · token adoption 87% → 89%

2. Point --color-bg-surface at the 96 inline #141414 uses
   Bypass · single find-and-replace · token adoption 89% → 93%

3. Add semantic roles for the 5 primitives read in components
   Bypass · 63 call sites · closes the tier boundary, enables the lint rule
```

Gap remediations rank above bypass remediations at equal movement, because a gap regenerates bypasses until it is closed.

Name the projected score honestly. A remediation that closes 41 of 2,461 values moves the number two points, and claiming more is the fastest way for the next audit to be ignored.

## Writing the findings

- **Name the value and the token together.** "Hardcoded color" is not actionable. "`#141414` where `--color-bg-surface` exists" is.
- **Never call drift a failure.** Report the rate against the baseline. A team that shipped six features and lost two points on component adoption is not regressing, and telling them they are ends the conversation.
- **Report a clean pass as a clean pass.** "No tier violations across 1,284 files" is a result worth writing down, and it is the row that makes the rest of the table credible.
- **Keep exceptions visible.** A confirmed exception sits below the table as context, so the next audit does not re-report it as a finding.
