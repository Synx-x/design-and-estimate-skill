<!-- plain: off -->
<div align="center">
  <h1>design-and-estimate</h1>
  <p><strong>Turn a feature request into a proposal that names real files, real layers and honest hours.</strong></p>
  <p>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
  </p>
</div>

`design-and-estimate` is an agent skill that writes a Design & Estimate (D&E) proposal for a new feature. It reads your repo first, so the document uses your own project names, folders and conventions. It asks once whether you want a plain-English summary or a detailed technical breakdown, then writes a `.txt` file and opens it.

```bash
npx skills add https://github.com/Synx-x/design-and-estimate-skill -g -y
```

Or copy `skills/design-and-estimate` into `~/.claude/skills/`. Then ask your agent for a proposal ("draft me a D&E for a customer export screen", "estimate this feature", `/design-and-estimate`).

## Why this skill

Estimates written from memory carry invented layers and wrong file paths. A reviewer spots the first bad path and stops trusting the rest. This skill researches the repo before it writes a word, and marks anything it cannot confirm as `[CONFIRM: ...]` instead of guessing.

## What the agent does

| Step | What happens |
|------|--------------|
| Gather | Reads the ticket, doc URL or description you give it, and asks one question if nothing is clear |
| Discover | Maps layers, screen and endpoint conventions, data entities, authorization, navigation, migrations and tests, using a code-exploration subagent |
| Ask | Asks Summary or Detailed every run, unless you already named the style |
| Write | Fills the matching template from the discovery, with ETAs sized line by line |
| Export | Writes `[FeatureName]_DE.txt` to the scratchpad, opens it and sends it to you |

## Two output styles

| Style | For | Contains |
|-------|-----|----------|
| Summary | Clients and non-technical stakeholders | Phase-grouped plain English, one sentence per deliverable, an ETA under each. No paths or class names |
| Detailed | Developers and internal planning | Structure by layer with exact paths, endpoints, data changes, navigation and authorization, infrastructure, a phase table and unresolved decisions |

## Estimate calibration

Each deliverable line is sized on its own, and the total is the sum of the lines.

| Work | AI-accelerated | Manual |
|------|----------------|--------|
| One screen or one service in an established area | 1 - 4 hr | 0.25 - 1 day |
| A screen plus its backend | 3 - 6 hr | about 1 day |
| A large module or external integration | 4 - 5 hr | about 1 day on the line |

## Guarantees

- The skill never picks the output style for you.
- It never invents a file path. Unknowns become `[CONFIRM: ...]`.
- A Summary contains no paths, class names or table names.
- A missing layer is recorded as a finding, not described as if it existed.
- Unresolved decisions appear as declarative either/or statements in the Detailed style.

## Requires

- An agent host that can read files and run shell commands.
- Optional: a connector for your ticket tracker (ClickUp, Jira, Azure DevOps, Linear, GitHub) when requirements live in a ticket.
- Optional: `references/known-stacks.md`, where you record shortcuts for repos you have already mapped. Discovery always wins over it.

## Repository

- `skills/design-and-estimate/SKILL.md`: the instructions the agent follows
- `skills/design-and-estimate/references/known-stacks.md`: optional per-repo shortcuts, shipped as a template
- `skills/design-and-estimate/evals/evals.json`: trigger and no-trigger prompts
- `LICENSE`: MIT
