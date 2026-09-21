# Xiangqi Claude Setup — Inventory (as of 2026-09-21)

This is the raw inventory that the shared repository was derived from. It records what exists in
the two Xiangqi repositories today, what each piece depends on, and the decision taken for each:
**share** (generalised into this repo), **keep** (stays project-specific), or **drop**.

Sources inspected: `xiangqi-client/.claude/`, `xiangqi-server/.claude/`, both `CLAUDE.md`
files, both PR templates, `.github/workflows/`, `dev-metrics-dashboard/`, the per-project Claude
memory, the global `~/.claude/CLAUDE.md`, MCP registrations in `~/.claude.json`, and git history
of the `.claude/` directories.

---

## 1. Context files

| File | Repo | Purpose | Project-specific content | Decision |
| --- | --- | --- | --- | --- |
| `.claude/CLAUDE.md` | client | Environment warning, stack, architecture map, code rules, NEVER list, ticket lifecycle table (Jira IDs), AI-contribution rules, Confluence sync, "before writing code" checklist | Everything except the *shape* | **Share the shape** as `templates/CLAUDE.md.template`; content stays per project |
| `.claude/CLAUDE.md` | server | Same shape + a long production-infrastructure section, test-safety rules (`ENVIRONMENT=testing`), migration patterns | All content | Same as above |
| `.claude/coding-standards.md` | client | 8 "core AI engineering rules" (never assume architecture, prefer existing patterns, hallucination prevention, verification requirement…), then React/Redux/styled-components/i18n/ESLint/Sonar specifics, PR-review lessons | Sections 2+ | **Share the 8 core rules** (they are stack-agnostic) as the top of `templates/coding-standards/*.md`; stack sections become example templates |
| `.claude/coding-standards.md` | server | Python/Flask/SQLAlchemy/Redis/Celery equivalents | All | Same treatment → `templates/coding-standards/backend-python.md` |
| `.claude/plans/_template.md` | client | Plan skeleton: Goal, Approach, Files, Redux/Saga impact, i18n, Risks, Regression Test rule, Out of Scope, Done When, AI Failure Points | Redux/i18n headings | **Share**, with the stack-specific headings replaced by a generic "Architecture impact" table |
| `.claude/plans/XQ-*.md` (62 files) | client | One plan per ticket, Jun–Sep 2026 | All | Keep. Used as evidence in the journey doc |
| `docs/superpowers/{plans,specs,runbooks}` | server | Same idea, superpowers-style naming | All | Keep |
| `.claude/ai-failures.md` | client | Cross-ticket log of AI mistakes, bucketed by area | Entries | **Share the format** as `templates/ai-failures.md` + `ai-failure-log` skill |
| `.claude/confluence-map.json` | both | file → Confluence pageId map for doc sync | All IDs | Keep; the *mechanism* is documented as an optional `docsSync` config |
| `.claude/settings.local.json` | client | Personal permission allowlist | Personal | Drop; a starter allowlist ships as `templates/settings.json` |
| `.github/PULL_REQUEST_TEMPLATE.md` | both | Jira link, summary, AI Contribution Checklist table, Planned/Actual SP | Weights differ (client 8 rows, server 9 rows) | **Share** as `templates/PULL_REQUEST_TEMPLATE.md`, weights driven by config |
| `~/.claude/CLAUDE.md` | global | mem0 usage rules | Personal | Drop |

## 2. Commands

| Command | Repo | Purpose | Inputs | Outputs | MCPs / tools | Project assumptions | Reusable? | Decision |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `/pick-ticket` | client | List my "Selected for Development" tickets on the active release, ask which to work on, hand off | none | table + handoff to `/start-ticket` | Jira MCP (JQL) | `project = XQ`, status name, active-release heuristic, Confluence KB caveat | High | **Share** — filter comes from config |
| `/start-ticket` | both | Show ticket, settle Planned SP (ask, never invent), transition to In Progress, cut branch | ticket key [SP] | Jira writes, branch | Jira MCP (get/edit/transition), git | `customfield_10010`, transition `31`, `<Name>/XQ-1234` branch, base `develop` | High | **Share** — IDs from config |
| `/analyze-ticket` | both | 10-point (client) / 8-point (server) technical impact analysis, no code | ticket key | analysis | Jira MCP | Stack-specific dimensions | High (shape) | **Share** — generic dimensions + hook into project standards |
| `/plan` | client | Gate on ticket state, run analyze, scaffold plan from template, ask sign-off | ticket + description | plan file | Jira MCP (indirect) | `.claude/plans/`, `XQ-` prefix | High | **Share** — paths from config |
| `/review-pr` | both | Senior-engineer review of current diff with fixed checklist and 🔴🟡🟢 severity | none (diff) | findings | git | Stack checklist (13 client / 14 server items) | High (shape) | **Share** as `/self-review` — generic rubric + project standards; stack items live in the project's coding standards |
| `/perf-check` | both | Performance-focused pass over the diff | none (diff) | findings with impact estimate | git | 75k games/day, React/Flask specifics | Medium | **Share** — generic checklist; stack specifics from project standards |
| `/create-pr` | both | Diff → PR body contract → AI Contribution table → push → `gh pr create` → Jira → Code Review → assignee/labels/reviewers → Confluence sync | [ticket] [title] | PR URL, table, sync report | Jira MCP, `gh`, Confluence MCP | Reviewer pool (3 named people), labels, `develop`, transition `51`, Confluence map | High (shape) | **Share** — every project fact moves to config |
| `/ai-contribution` | both | Print the AI Contribution table for the thread without creating a PR | [ticket] | table + honesty statement | none | Weights | High | **Share** — weights from config |
| `/pr-review-comments` | client | Walk unresolved PR review threads, implement, resolve with user approval | PR URL | code changes, resolved threads | `gh` (REST + GraphQL) | none — already generic | High | **Share** as `/pr-comments` (trimmed) |
| `/write-tests` | server | Write tests for changed code following repo conventions | none | tests | none | pytest/Factory Boy | Medium | Fold into `testing-standards` skill + `/implement`; no standalone command (repos differ on test policy) |
| `/request-translations` | client | Find English-only i18n keys in a PR, post a Jira comment asking for translations | ticket + PR URL | Jira comment | Jira MCP, `gh` | Four locale files, a named translator | Low | **Keep** project-specific |
| `/apply-translations` | client | Parse a Jira comment with translations and write locale files in place | Jira comment URL | file edits | Jira MCP | Locale file quirks | Low | **Keep** |
| `/add-theme` | client | Orchestrate a new visual theme end-to-end | theme inputs | code | Figma MCP | Entirely Xiangqi | None | **Keep** |

Observed but *not* present as commands: `/plan-review` (plan sign-off happens conversationally),
`/run-review`, release commands. Self-review is `/review-pr` + `/perf-check` run before opening
the PR; there is no separate "self review" step written down on the client, though the server's
`pre-review-checklist` skill fills that role.

## 3. Skills

| Skill | Repo | Purpose | Reusable? | Decision |
| --- | --- | --- | --- | --- |
| `pre-review-checklist` | server | Mechanical hygiene pass before human review (dead code, misplaced logic, `.all()`, duplicated cache-key calls, run lint/tests twice, get a subagent review, verify reviewer claims) | High (items 1, 6 are generic; 2–5 are Flask-specific) | **Share the generic half** inside `review-standards`; keep Flask specifics in the project |
| `xiangqi-knowledge` (router) + `xiangqi-board`, `xiangqi-engine`, `xiangqi-rules`, `xiangqi-bot`, `xiangqi-trade`, `xiangqi-testing` | server | Domain knowledge (game rules, UCI, bot logic, test patterns) | None | **Keep** — but the *pattern* (a router skill + focused domain skills with trigger keywords) is documented in the Skills Catalog as the recommended way to capture domain knowledge |

The client has no `skills/` directory; its standards live in `coding-standards.md` and are
loaded via `CLAUDE.md`. Both repos also have the `superpowers` plugin installed at user level
(brainstorming, writing-plans, TDD, systematic-debugging, code review request/receive).

## 4. Agents

| Agent | Repo | Purpose | Reusable? | Decision |
| --- | --- | --- | --- | --- |
| `theme-creator` | client | Build a colour palette + registration for a new theme | None | **Keep** |
| `board-creator` | client | Register dropped board assets and wire mappings | None | **Keep** |

No generic agents exist. The server's pre-review skill *dispatches* the superpowers
code-reviewer subagent. The shared repo adds two justified generic agents (`plan-reviewer`,
`code-reviewer`) because their value is an independent context, which a command cannot give.

## 5. MCP and integrations

| Integration | How it is used today | Mandatory? | Fallback today |
| --- | --- | --- | --- |
| **Jira** (`atlassian-rovo-mcp`, user-scope) | `searchJiraIssuesUsingJql`, `getJiraIssue`, `editJiraIssue` (SP), `transitionJiraIssue`, `addCommentToJiraIssue`; Confluence page create/update for doc sync | For `/pick-ticket`, `/start-ticket`, transitions in `/create-pr`; optional elsewhere | `/apply-translations` documents a paste fallback; other commands stop and say the MCP is needed |
| **GitHub** | Entirely through the `gh` CLI (`pr create/edit/view`, `api`, GraphQL for thread resolution). No GitHub MCP is registered | `gh` is required for `/create-pr`, `/pr-comments` | none |
| **Figma** (`figma`, project-scope on client) | `get_design_context`, `get_screenshot`, `get_variable_defs` — referenced in 7 plans and `/add-theme`; used ad hoc during analysis/planning, not by a command | Optional | Screenshots pasted by the user |
| **Confluence** (same Atlassian MCP) | Auto-sync of plans/agents/commands on PR creation via `confluence-map.json` | Optional | skip |
| **SonarQube** | CI only (`build.yml` runs the scanner on every PR); commands reference its rules by ID | CI | n/a |
| **mem0** | Personal cross-project memory (user-level) | Optional | n/a |
| **Jira custom fields** | Planned SP `customfield_10010` (written by `/start-ticket`), Actual SP `customfield_10833` (humans only), AI Contribution % `customfield_11729` (read by the dashboard; **no command writes it** — copied from the PR by hand) | — | — |

## 6. Measurement

`dev-metrics-dashboard` (separate Node project) pulls Jira (SP, AP, AI %) and GitHub PRs
(evidence only) into an HTML dashboard: release summary, progress over time, In Progress → Code
Review cycle time, per-developer delivery, all tickets. Deliberately no composite score or
ranking (v1 had one; it was discarded after testing against real data). The AI % on the Jira
ticket is entered manually from the PR table.

## 7. Workflow actually in use (observed)

```
/pick-ticket ──▶ /start-ticket ──▶ /plan (runs /analyze-ticket) ──▶ sign-off (chat)
      ──▶ implement (conversational, plan open) ──▶ /review-pr + /perf-check (self-review)
      ──▶ /create-pr (AI table, Jira → Code Review, labels/reviewers, Confluence sync)
      ──▶ human review on GitHub ──▶ /pr-review-comments ──▶ merge (human) ──▶ Jira AI % by hand
      ──▶ dashboard
```

Evidence: 62 plan files on the client (Jun 6 → Jul 19 → Aug 30 → Sep 7 per month), 13 of them
with AI Failure Points logged (46 entries), 6 revisions of `/create-pr` (the most-iterated
command), and the `Ticket Lifecycle` table in both `CLAUDE.md` files. Server plans live under
`docs/superpowers/` instead, and the server has `/write-tests` because it does write tests.

## 8. Gaps found

1. No written self-review step on the client (the server has one as a skill).
2. No plan review with independent context — sign-off is the author reading their own plan.
3. Nothing writes the AI % back to Jira; it is re-typed by hand (source of drift).
4. `/analyze-ticket` never looks at linked issues or related recent changes.
5. No release-notes or release-readiness support; releases are done outside Claude.
6. The AI failures log is only filled on ~20 % of plans; there is no command to make logging a
   two-second action.
7. Six near-duplicate commands are maintained twice (client and server copies drift — e.g. PR
   title style `XQ-1: x` vs `XQ-1 - x`, different weight tables).
8. Every project fact (field IDs, reviewers, labels, branch names) is inlined into prompts, which
   is exactly what stops the commands being copied to another project.
9. Onboarding a new repo means hand-copying and hand-editing ~10 files.

The shared repository addresses 1–2 and 4–9 directly; 3 is made an opt-in config
(`aiContribution.writeToTracker`).
