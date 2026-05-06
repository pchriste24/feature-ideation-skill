# feature-ideation

An agent skill that analyzes a local codebase and generates a ranked list of product feature ideas grounded in the actual code, data models, and inferred user workflows — not generic startup advice.

## What it does

The skill reads your repository, builds a mental model of what the product is and who uses it, imagines real usage sessions to find friction points, and produces a prioritized `FEATURE_IDEAS.md` with concrete, actionable ideas. Every suggestion is anchored to something in the code — a model, a route, a dependency, a gap in an existing flow.

Output is organized into three tiers:

- **Quick wins** — high value, low effort; shippable in days or weeks
- **Substantial features** — the meat of the next quarter
- **Ambitious bets** — bigger swings worth considering

## Usage

Install the skill, then ask Claude things like:

- "What should I build next?"
- "Give me feature ideas for this product"
- "What's this product missing?"
- "Ideas for v2"
- "What would make this better?"

Optionally, name a competitor for contrast analysis:

> "What features does Notion have that this product is missing?"

No competitor input is required — the codebase itself is the primary source.

## Installation

```bash
npx skills add pchriste24/feature-ideation-skill -g
```

## Output

Saves `FEATURE_IDEAS.md` to the repo root with this structure:

```
# Feature Ideas

Product: ...
Primary users (inferred): ...

## Quick wins
## Substantial features
## Ambitious bets
## Ideas considered but cut
```

## Requirements

- Claude Code (the skill uses file-reading tools to explore the local repo)
- A codebase Claude has access to via the filesystem

## License

MIT