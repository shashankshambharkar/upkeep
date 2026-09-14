# AGENTS.md

This file is the single source of guidance for coding agents working in this repository. `CLAUDE.md` imports it and adds nothing but Claude Code specifics, so put repository facts here and do not maintain a second copy.

## What this repository is

A collection of agent skills for design system operations: token architecture, component API design and adoption auditing. It ships two ways, via `npx skills add shashankshambharkar/drift` and as the Claude Code plugin `drift` served by the marketplace in this same repository. It is documentation-only, with no build, lint, or test tooling.

`.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` define the plugin and its marketplace. Both are named `drift`, so plugin users invoke skills as `/drift:design-system-audit` while skills-CLI users invoke `/design-system-audit`. Skills are discovered from `skills/` automatically, so adding one needs no manifest change.

Bump `version` in `plugin.json` in the same commit as any change under `skills/`. That number is the only signal plugin users update on, and a change shipped without a bump reports "already at the latest version" and never reaches them. Run `claude plugin validate .` after touching either manifest.

`opencode.json` registers `skills/` under `skills.paths` so opencode loads the collection while this repository itself is open, which is for working on the skills rather than distributing them.

## Scope

This collection owns design system *operations*: the decisions that keep a system usable after it exists. It does not own interface craft. Typography, color values, layout, motion aesthetics and accessibility have a good home in the `better-*` family, and duplicating a rule that lives there makes both copies worse.

The test for a new skill: does it answer a question a design system owner has on a Tuesday, about a system that already exists? Token naming, yes. Deprecation, yes. Which easing curve feels right, no.

## Structure

Each skill lives in `skills/<skill-name>/`, with `SKILL.md` as the entry point, supporting files in `references/` and a harness config in `agents/openai.yaml`.

Skills come in two shapes. A domain skill holds knowledge: what is true about tokens or about component APIs. A verb skill holds a procedure: audit this codebase. A procedure sitting inside a domain skill is a candidate for extraction, and a domain rule sitting inside a verb skill belongs to its owner instead.

Every `SKILL.md` carries:

- **Frontmatter** with `name` matching the directory and `description`.
- **A plain-name H1** and a two-sentence opener saying what the skill is and what it does. Not what the domain means or why it matters: an agent does not need motivating.
- **A calibration section**, with a heading that carries its own point (`Exact, or say you did not check`), saying how hard to press. Which values are exact rather than approximate, what counts as a finding versus a preference, when the right answer is to write nothing. A skill that lists rules without saying how hard to press leaves that to chance.
- **Headings that carry the point**, in sentence case. `Count roles, not values`, not `Auditing`. Number them only where the steps genuinely run in order, as `design-system-audit` does. Numbering flat reference implies a sequence that is not there and makes every insertion a renumber.
- **A hand-off line** naming the sibling skills that own adjacent topics, by skill name in backticks.
- **A `## Before you finish` table**, two columns. The left column is the detection pattern, which is what a principle statement does not give you. The heading names the moment on purpose: `Common mistakes` is a label an agent reads past while orienting.
- **A `## Reporting` section** carrying that skill's severity ladder, its verification checks and its output format.

Supporting files in `references/` carry depth beyond the principle statements: procedures, code patterns, lookup tables. Link each from the principle that needs it, so the link sits where the agent lands. A principle states the rule and links out for the recipe. It never restates the reference file in shorter form, and the reference file never restates the principle in longer form.

Each rule lives in exactly one skill. Other skills point to it by skill name in backticks, never by cross-skill relative link, because each skill directory ships on its own. That standalone requirement is also why the three `## Reporting` sections overlap, and that overlap is the price of a skill that works when installed alone.

Point at a principle by its heading in bold (**Count roles, not values**), never by its number. A numbered reference breaks silently the moment a principle is inserted above it, and nothing in the file fails when it does.

## Invocation

A user-invoked skill may invoke model-invoked skills, but it can never reach another user-invoked skill.

- `design-system-audit` is the only user-invoked skill. It carries `disable-model-invocation: true` in its frontmatter **and** `policy.allow_implicit_invocation: false` in its `agents/openai.yaml`. Those are the Claude Code and Codex halves of one switch and must be set together, or the skill behaves differently per harness.
- It is user-invoked because auditing a whole codebase is never something to start on someone's behalf. It runs dozens of greps, writes a report sized for a planning meeting and asks scoping questions only a person can answer. An agent firing it after every component it touched would produce work nobody asked for.
- `design-tokens` and `component-api` are model-invoked, because something must reach them. They are also where `design-system-audit` routes every finding.

## Rule ownership

| Skill | Owns |
| --- | --- |
| `design-tokens` | Tier architecture and the component-may-not-read-a-primitive boundary, the role inventory across every token category, semantic naming grammar, the collapse procedure, theme completeness and the Figma variable mirror |
| `component-api` | Variant axes and invalid combinations, prop naming vocabulary, the configuration-to-composition threshold, slots and compound components, controlled and uncontrolled state, escape hatches and the deprecation path |
| `design-system-audit` | Audit scope resolution, the eight detection passes, the gap / bypass / exception classification, the two adoption scores and their arithmetic, remediation ranking and the report format. Owns no domain rules and issues no verdict |

When a concern crosses skills, keep the rule in the owner above and let the other name only the handoff:

- `design-tokens` owns what a token should be named and where it may be read. `design-system-audit` owns finding the places that broke those rules, and names the finding in `design-tokens` vocabulary.
- `component-api` owns whether a prop should exist. `design-system-audit` owns counting how many call sites set it.
- `design-tokens` owns component-scoped tokens as a tier. `component-api` owns which variant cell binds to which token, and states no naming rule of its own.
- Color values, ramp construction and contrast measurement belong to `better-colors` where that skill is installed. `design-tokens` owns the architecture those values sit in and never restates a ramp rule.

## Authoring conventions

- Principles are prescriptive and specific: exact property names, exact values, exact commands. Not vague advice.
- Match the degree of prescription to the decision. A tier boundary is unconditional. A heuristic names its context and its escape conditions before giving exact values.
- Skills instruct agents to match the target project's existing conventions rather than impose one. A project on `variant` should not be told to rename to `appearance`; it should be told to pick one and grep the other to zero.
- Every command in a reference file must be runnable as written, against a real project, with its assumptions stated above it. A command that needs adjusting says which part to adjust.
- Frontmatter `description` is how a skill gets found, and it is one or two plain sentences saying what the skill does for the user. It loads on every turn, so it earns harder pruning than the body. No trigger list: a keyword pile is a worse match signal than a clear sentence and goes stale the moment the skill's scope moves. The wording matches the skill's line in `README.md`, so changing one means changing both.
- A skill's name appears in three places: its directory, its frontmatter `name` and `display_name` in its `agents/openai.yaml`. Renaming means changing all three, then grepping for the old name to confirm nothing survived.
- Prefer counts and lists that cannot go stale. Say "every skill in this repository" rather than a number the next skill invalidates.
- Straight quotes, sentence-case headings. No em dashes and no parentheses or mid-sentence colons standing in for one: end the sentence or use a comma. En dashes are for numeric ranges only.
- No serial comma. Write `tokens, components and drift`, never `tokens, components, and drift`. The comma stays where `and` joins two clauses, and where dropping it would swallow an appositive.

Four checks after an edit, since prose drifts back toward the mean:

- **No sentence over 30 words**, counting a code span as one word. A ceiling, not an average. What makes a file hard to read is the individual 40-word sentence carrying four clauses, so split those and leave the rest alone.
- **A description that matches its `README.md` line.** Two wordings of one skill is one skill described twice.
- **One statement of each rule.** Before adding a sentence, check whether the file already says it somewhere else. The reflex to restate a boundary for clarity is how a mistake table ends up repeating the principle above it.
- **A pruning pass, not a word ceiling.** Read each sentence and ask what it changes. A sentence that cannot be restated as an instruction, a fact, or a number is cut. A sentence that could appear unchanged in another project's docs says nothing about this one. Prose about this repository's own filing decisions belongs in this file, never in a skill.
