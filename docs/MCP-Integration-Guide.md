# MCP Integration Guide

The shared commands need exactly one hard dependency — the `gh` CLI for GitHub — and treat
every MCP as optional with a documented fallback. This guide says what each integration gives,
which commands use it, what it needs, and what happens without it.

## Summary

| Integration | Transport | Used by | Mandatory? | Without it |
| --- | --- | --- | --- | --- |
| **Jira** (Atlassian MCP) | Remote MCP (`mcp.atlassian.com`) or claude.ai connector | pick-ticket, parallel-tickets, start-ticket, analyze-ticket, create-pr, release-notes, setup-workflow | Required for `/pick-ticket`; optional elsewhere | Paste fallback; tracker writes listed as "to do by hand" |
| **Confluence** (same Atlassian MCP) | as above | create-pr (docs sync) | Optional (`docsSync.type`) | Skipped |
| **GitHub** | `gh` CLI (no MCP) | create-pr, pr-comments, release-notes, setup-workflow | Required for those commands | Command stops at the GitHub step and says so |
| **Figma** | Remote MCP (`mcp.figma.com`) or claude.ai connector | analyze-ticket (and plan via analyze) | Optional (`figma.enabled`) | Ask for screenshots; never describe an unseen design |
| **CI quality gate** (SonarQube etc.) | CI only | none directly; rules referenced from coding standards | — | — |

Tool names are matched by **suffix** (`getJiraIssue`, `get_design_context`) because the prefix
depends on how the server was registered (`mcp__atlassian-rovo-mcp__…`,
`mcp__claude_ai_Atlassian_Rovo__…`).

## Jira

**Provides:** ticket text (the ground truth for analysis and estimates), status, issue type,
fix versions, linked issues, custom fields (story points, AI %), transitions, JQL search,
comments.

**Tools used:** `getAccessibleAtlassianResources`, `getVisibleJiraProjects`,
`searchJiraIssuesUsingJql`, `getJiraIssue`, `getTransitionsForJiraIssue`, `transitionJiraIssue`,
`editJiraIssue`, `getJiraIssueTypeMetaWithFields`, `addCommentToJiraIssue` (project commands
only).

**Permissions:** the engineer's own Atlassian account via OAuth — the commands act as the
user. Writes are limited to: story points (after a human answers), two status transitions,
optionally the AI % field. Nothing else is ever written by a shared command.

**Configuration:** `tracker.*` in `.claude/workflow.json`. Field and transition ids are
instance-specific and must be verified — `/setup-workflow` does this by calling
`getTransitionsForJiraIssue` and `getJiraIssueTypeMetaWithFields` and asking the user to
confirm. Never copy ids from another project.

**Setup:** `claude mcp add --transport http atlassian-rovo-mcp https://mcp.atlassian.com/v1/mcp`
(user scope so it follows the engineer across repos), then `/mcp` to authenticate. Or the
claude.ai Atlassian connector.

**Known limitations (observed):**
- Search payloads are large; always restrict `fields` and extract with `jq` from the saved file.
- The bulk search does not return status history; cycle-time metrics need the per-issue
  changelog endpoint (the metrics dashboard does this outside Claude).
- Confluence spaces exposed depend on the connector's grants; a command must not assume a
  knowledge-base space exists.
- OAuth sessions expire; a non-interactive session cannot re-authenticate.

## GitHub (via `gh`)

**Provides:** PR create/edit/view, labels, reviewers, review threads (GraphQL for resolution
state), file contents at a ref (`gh api repos/…/contents`), compare/merge-base.

**Why no MCP:** `gh` is already authenticated on every engineer's machine, supports GraphQL,
and makes the audit trail the engineer's own account. A GitHub MCP would add a second
credential for no capability the commands need.

**Permissions:** the engineer's `gh auth` token. Commands never force-push, never merge, never
delete branches.

**Configuration:** `github.*` — repo (optional, derived otherwise), labels, reviewer pool,
policy, confirm-before-create.

**Limitations:** requesting review from the PR author fails the whole call (the skill filters
the author out); renamed organisations make `origin` redirect — prefer a URL the user gave.

## Figma

**Provides:** design context (layout, text, components), screenshots, variable/token
definitions, Code Connect mappings.

**Used for:** grounding UI tickets in the real design during `/analyze-ticket`, recording the
node links in the plan, and comparing rendered output at self-review. Shared commands never
write to Figma.

**Configuration:** `figma.enabled: true`. **Permissions:** view access to the design files.

**Limitations:** node-level reads only — ask for the specific node; labels in the design can
disagree with swatch values (trust the swatch); design and ticket can disagree (raise it,
don't pick).

## Confluence (optional docs sync)

**Provides:** page create/update so plans and command docs are readable by people who never
open a diff.

**Mechanism:** a file → `pageId` map (`docsSync.map`) keyed by file name, with parent page ids;
`/create-pr` updates known pages in place and creates+records new ones. Titles must be unique
per space — prefix by repo if two repos share a space.

**Limitations:** Markdown → Confluence conversion is lossy for tables and code; keep synced
docs simple. If the map drifts (page renamed/deleted), fix the map rather than letting a
duplicate page be created.

## Adding an integration

1. Put the rules in one skill (`<name>-integration/SKILL.md`): tool map, what it provides,
   fallback.
2. Add a config section with `enabled`/`type` and `null` defaults.
3. Commands check the config, call the skill, and skip with a message when disabled.
4. Document it here with the same five headings: provides, used by, permissions,
   configuration, limitations.
