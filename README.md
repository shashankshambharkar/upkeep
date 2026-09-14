# Upkeep

Agent skills for running a design system, not just building one.

[![skills.sh](https://skills.sh/b/shashankshambharkar/upkeep)](https://skills.sh/shashankshambharkar/upkeep)

A design system is maybe ten percent design and ninety percent upkeep: deciding what a token is for, keeping a component's API stable while its internals change and finding out what the codebase actually renders versus what the library documents. These skills cover that part.

They come out of running a production design system at [Upstox](https://upstox.com), where a token set went from 240 declarations to 27 roles without anything on screen changing.

## Skills

- [**design-tokens**](skills/design-tokens/SKILL.md): Helps you design, audit and shrink a design token system. Covers **tier architecture**, **naming grammar**, the **role inventory** a system needs, **collapsing** a bloated set and keeping every token **resolvable in every theme**.
- [**component-api**](skills/component-api/SKILL.md): Helps you design and review the public API of design system components. Covers **variant axes**, **prop naming**, when **configuration should become composition**, **controlled state**, **escape hatches** and **deprecating** a prop without breaking consumers.
- [**design-system-audit**](skills/design-system-audit/SKILL.md): Audits how well a codebase actually uses its design system and reports what drifted. Finds **hardcoded values**, **tier violations**, **dead tokens**, **duplicate components** and props only one screen sets, then **scores adoption** and routes each finding to the skill that owns the fix. User-invoked.

`design-tokens` and `component-api` hold the rules. `design-system-audit` holds a procedure and owns no rules of its own: every finding it reports names the skill that diagnoses the fix.

## Install

```bash
npx skills add shashankshambharkar/upkeep
```

## Claude Code plugin

```text
/plugin marketplace add shashankshambharkar/upkeep
/plugin install upkeep@upkeep
```

## What each skill will not do

`design-tokens` does not pick colors. Ramp construction and contrast measurement belong to [`better-colors`](https://github.com/jakubkrehel/skills), and this skill owns the architecture those values sit in.

`component-api` does not review visual or interaction craft. Those belong to `better-ui` and `better-accessibility`.

`design-system-audit` does not fix anything and issues no verdict. It reports two scores, a ranked table and the three remediations that would move the score most, then stops.

## Author

[Shashank Shambharkar](https://shashanks.design), design systems designer. Pune, India.

MIT.
