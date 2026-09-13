---
name: design-system-audit
description: Audits how well a codebase actually uses its design system and reports what drifted. Finds hardcoded values, tier violations, dead tokens, duplicate components and props only one screen sets, then scores adoption and routes each finding to the skill that owns the fix.
disable-model-invocation: true
---

# Design system audit

This skill measures the distance between the design system a team documents and the one their codebase renders. The output is an adoption score and a findings table, both derived from commands that ran, not from reading the component library and assuming.

It counts rather than judges. A finding here is a divergence a command found, named in the vocabulary of the skill that owns the fix. Designing the token architecture is `design-tokens`, designing the component API is `component-api`, and this skill decides neither.

Drift is normal and a zero-drift report usually means the audit missed a directory. The useful question is not whether a system drifted but whether it is drifting faster than it is being adopted, which is why the score is reported alongside the count.

## Measured, or not reported

Every finding carries the command that produced it and the paths it ran over. A number nobody can reproduce is an opinion with a decimal point, and a system owner is about to take this report to a planning meeting.

Where a check could not run, report the role as `Not verified` and say why. An audit that quietly drops the checks it could not perform reads as a clean bill of health, which is the one outcome it must never fake.

## 1. Scope before counting

Establish four things, in this order, and stop to ask where any is ambiguous.

- **The system's source.** Where the tokens are declared and where the components are exported from. The exported entry point, not the source directory: an internal prop is not adoption surface.
- **The consuming surface.** The directories that should be using the system. Exclude the system's own source, tests, generated files and vendored code, and say what you excluded.
- **Deliberate exceptions.** Marketing pages, an in-progress migration and a legacy area nobody is funding are all legitimately outside the system. Ask, rather than reporting a team's own decision back to them as a defect.
- **The baseline.** Whether a previous audit exists. A score means little alone and a lot next to last quarter's.

Where the repo holds several apps, audit one and say so. A combined score across a new app and a legacy one describes neither.

## 2. Inventory both sides

Extract the declared tokens and the exported components before looking for anything. The counts are the denominator every later number divides into.

Produce three lists: every declared token with its tier, every exported component with its props and every role from the `design-tokens` role inventory marked covered or missing. [detection.md](references/detection.md) has the extraction commands for CSS custom properties, a JS token object and a components barrel file.

## 3. Run the passes in order

Eight passes, each producing one finding type. Run them all, because the interesting result is usually the relationship between two of them. [detection.md](references/detection.md) holds the command for each.

| Pass | Finds | Owner |
| --- | --- | --- |
| Hardcoded values | Colors, spacing and radii written inline where a token exists | `design-tokens` |
| Tier violations | Primitives referenced inside components | `design-tokens` |
| Missing roles | Values hardcoded because no token covers the role | `design-tokens` |
| Dead tokens | Declared, referenced nowhere | `design-tokens` |
| Theme holes | Semantic tokens that do not resolve in every theme | `design-tokens` |
| Shadow components | Local implementations of a pattern the system exports | `component-api` |
| Single-consumer props | Props exactly one call site sets | `component-api` |
| Bypassed imports | System components imported from source paths rather than the entry point | `component-api` |

A high hardcoded count next to a high missing-role count is one finding, not two: the system is incomplete and teams are routing around it. A high hardcoded count with full role coverage is a different finding entirely, and the remedy is not the same.

## 4. Classify by cause, not by count

Every finding gets one of three causes, and the cause decides who acts.

- **Gap.** The system has no answer, so a team invented one. The system team owns it. A missing role and a shadow component that no system component covers are both gaps.
- **Bypass.** The system has an answer and the code did not use it. The consuming team owns it. A hardcoded value where the token exists is the clearest case.
- **Exception.** A deliberate divergence, confirmed in step 1 or documented in the code. Nobody owns it, and it belongs in the report as context rather than as a finding.

Classify by reading the call site, not by pattern-matching the name. A hardcoded `#fff` inside a component that takes a color prop may be a default rather than a bypass.

Report gaps first regardless of count. Twelve bypasses are a cleanup, and one gap is why the next twelve will happen.

## 5. Score adoption, and show the arithmetic

One number, defined the same way every quarter so the trend means something:

```
adoption = tokenised values / (tokenised values + hardcoded values)
```

Counted over the consuming surface only, with exceptions excluded from both terms. Report it beside the three counts that explain it, and never as a grade. See [report.md](references/report.md) for the full table.

Report component adoption separately rather than blending it in, because the two move independently and a single blended number hides which one moved.

## 6. Report, route and stop

The deliverable is one table, gaps first, then bypasses by count descending. Each row names the owning skill so the fix starts in the right place. [report.md](references/report.md) holds the format and the summary block that goes above it.

Then stop. This skill fixes nothing: on a request to fix, follow the owning skill's rules and re-run the affected pass to confirm the count moved.

Name the top three remediations by count and say what each would move the score to. A report that ends with 340 findings and no order is a report nobody starts.

## Before you finish

| Mistake | Fix |
| --- | --- |
| A count with no command behind it | Run the pass, or report the role as `Not verified` |
| Checks that could not run, silently dropped | List them as `Not verified` with the reason |
| Auditing the source directory as consuming surface | Exclude the system's own source, tests and generated files |
| Counting a team's documented exception as a defect | Confirm exceptions in step 1 and report them as context |
| A hardcoded value reported without checking a token exists for it | Check the role inventory first; no token means gap, not bypass |
| Reporting bypasses above gaps because there are more of them | Gaps first; they are why the bypasses keep arriving |
| One blended score across tokens and components | Two numbers, reported separately |
| A score with no denominator shown | Publish the arithmetic beside the number |
| A combined score across several apps | Audit one app per run and say which |
| Findings listed with no owner | Every row names `design-tokens` or `component-api` |
| A 300-row table with no ordering | Rank by count and name the top three remediations |
| Fixing findings during the audit | Report, then fix on request, then re-run the pass |
| Calling drift a failure | Report the rate against the baseline, not the raw count |

## Reporting

**Severity.** `HIGH` is a gap: the system has no answer and teams are inventing them. `MEDIUM` is a bypass with more than a handful of occurrences, or any tier violation. `LOW` is an isolated bypass, a dead token and a naming inconsistency.

**Verification.** Name the consuming surface, the excluded paths and the command per pass. Confirm a token exists before classifying a hardcoded value as a bypass. Resolve semantic tokens in every declared theme before reporting coverage. Grep the full surface, including stories and docs, before reporting a token dead.

**Format.** One table, gaps first, then bypasses by count descending:

| Cause | Finding | Count | Locations | Owner |
| --- | --- | --- | --- | --- |

`Locations` gives up to three `path/to/file:line` examples and a count for the rest. Above the table goes the summary block from [report.md](references/report.md).

This skill issues no verdict. It reports two scores, a ranked findings table and the three remediations that would move the score most. With nothing to report, publish the scores and the passes that ran anyway, because a clean audit is only meaningful next to the surface it covered.
