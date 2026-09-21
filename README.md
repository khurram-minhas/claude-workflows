# Arbisoft Claude Workflows

Shared Claude Code commands, skills and agents for the ticket-to-PR engineering lifecycle,
extracted from the Xiangqi team's setup and made project-independent. Install it as a plugin,
add one config file to your repository, and the same workflow runs against your Jira project,
your GitHub repo and your coding standards.

```
/pick-ticket ─▶ /start-ticket ─▶ /analyze-ticket ─▶ /plan ─▶ /plan-review ─▶ sign-off
  ─▶ /implement ─▶ /self-review (/perf-check) ─▶ /create-pr ─▶ review ─▶ /pr-comments ─▶ merge
```

Every step that changes a ticket, opens a PR or resolves a thread stops for a human first.

## Install

As a plugin (recommended — one copy, updates flow to everyone):

```
/plugin marketplace add arbisoft/arbisoft-claude-workflows      # or the internal git URL
/plugin install arbisoft-workflows@arbisoft-claude-workflows
```

For local testing: `/plugin marketplace add ./arbisoft-claude-workflows` then install as above
(loaded in place, edits picked up immediately).

Or copy `commands/`, `skills/`, `agents/` into a repository's `.claude/` — works, but you own
the updates.

## Set up a repository

```
/setup-workflow team      # or: /setup-workflow solo
```

It detects the stack, verifies Jira field and transition ids live (never guessed), writes
`.claude/workflow.json`, and scaffolds the plan template, failure log, coding standards and
PR template from `templates/`. Details: [docs/Configuration-Guide.md](docs/Configuration-Guide.md).

Prerequisites: `gh` authenticated; the Atlassian MCP connected (`claude mcp add --transport
http atlassian-rovo-mcp https://mcp.atlassian.com/v1/mcp`, then `/mcp`) if you use Jira;
the Figma MCP if you have designs. All optional except `gh` — see
[docs/MCP-Integration-Guide.md](docs/MCP-Integration-Guide.md).

## Layout

```
commands/     14 lifecycle commands — sequence only, no rules, no project facts
skills/       13 skills — the rules, each in exactly one place
agents/       plan-reviewer, code-reviewer — independent-context reviews
config/       workflow.defaults.json + workflow.schema.json
templates/    CLAUDE.md, coding standards (generic / React / Python), plan, PR, failure log, settings, mcp
examples/     single-engineer, frontend-react-project, backend-python-project, team-config
docs/         adoption guide, Xiangqi journey, catalogs, MCP + configuration guides, inventory
```

## Read next

- New to this? [docs/Arbisoft-Claude-Adoption-Guide.md](docs/Arbisoft-Claude-Adoption-Guide.md)
  — principles, two adoption models (one engineer / a team), roadmap, maturity model, coaching
  playbook.
- Why it looks like this: [docs/Xiangqi-AI-Journey.md](docs/Xiangqi-AI-Journey.md).
- What each piece does: [docs/Command-Catalog.md](docs/Command-Catalog.md),
  [docs/Skills-Catalog.md](docs/Skills-Catalog.md).
- What was inventoried and what stayed project-specific:
  [docs/Xiangqi-Setup-Inventory.md](docs/Xiangqi-Setup-Inventory.md).

## Non-negotiables baked into the commands

- Never move a ticket backwards. Never invent story points. Never write Actual SP.
- Failed tracker/GitHub calls are reported next to successes, never folded into them.
- AI contribution is an estimate: blank rows for activities that did not happen, one
  percentage, always labelled retrospective.
- Read the file before editing it; no new abstractions unless asked; verify before "done".

Contributing: [CONTRIBUTING.md](CONTRIBUTING.md).
