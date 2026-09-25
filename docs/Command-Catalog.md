# Command Catalog

Fifteen commands, one lifecycle. Each command has one responsibility and composes shared
skills instead of restating rules. Project facts come from `.claude/workflow.json`
(see [Configuration-Guide.md](Configuration-Guide.md)).

```
/setup-workflow (once)
   │
/pick-ticket ─▶ /start-ticket ─▶ /analyze-ticket ─▶ /plan ─▶ /plan-review ─▶ [human sign-off]
   ─▶ /implement ─▶ /self-review (+ /perf-check) ─▶ /create-pr ─▶ [human review] ─▶ /pr-comments
   ─▶ [human merge]            side commands: /ai-contribution · /log-failure · /release-notes

/parallel-tickets ─▶ one worktree + one session per ticket, each running /start-ticket ─▶ …
```

When installed as a plugin the commands may appear namespaced (`/arbisoft-workflows:plan`);
when copied into a repository's `.claude/commands/` they are bare (`/plan`). The text of each
command refers to the bare name.

## Lifecycle commands

| Command | Purpose | Inputs | Outputs | Writes to | Skills used | MCP / tools | Origin |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `/setup-workflow [solo\|team]` | Bootstrap a repo: detect stack, verify tracker ids, write config, scaffold templates | profile | `.claude/workflow.json`, plan template, failure log, standards, PR template, CLAUDE.md section | repo files | workflow-config, jira-tracker | Jira MCP (optional), `gh` | **New** — closes the "hand-copy 10 files" gap |
| `/pick-ticket` | List my ready tickets, choose one | — | table → handoff | nothing | workflow-config, jira-tracker, human-in-the-loop | Jira MCP (required) | Xiangqi `/pick-ticket`, filter now config-driven |
| `/start-ticket <KEY> [SP]` | Show ticket, settle SP with a human, → In Progress, cut branch | ticket | branch + report | tracker (SP, status), git | workflow-config, jira-tracker, git-standards, human-in-the-loop | Jira MCP (optional writes), git | Xiangqi `/start-ticket`, ids from config |
| `/parallel-tickets [KEY ...]` | Work 2–5 tickets at once: one worktree (detached at base) + one interactive Claude session per ticket | keys or multi-pick from queue | worktrees, launched sessions, report | git worktrees only | workflow-config, jira-tracker, parallel-worktrees, human-in-the-loop | git, Jira MCP (optional), `code` / `tmux` (optional) | **New** — parallel work without giving up any gate |
| `/analyze-ticket <KEY\|text>` | Grounded technical analysis, open questions, related work, risks, steps | ticket or pasted text | analysis | nothing | workflow-config, jira-tracker, figma-design, ai-failure-log, security-review, performance-review | Jira MCP (optional), Figma MCP (optional) | Xiangqi `/analyze-ticket`, generic layers + linked issues + failure-log lookup added |
| `/plan <KEY> <desc>` | Create the plan file from the template; ask for sign-off | ticket | plan file | `plans.dir` | workflow-config, planning-standards, testing-standards, human-in-the-loop | — | Xiangqi `/plan` |
| `/plan-review <KEY\|path>` | Independent challenge of a plan; proposed edits | plan | findings + edited plan | plan file (accepted edits only) | planning-standards, review-standards | `plan-reviewer` agent | **New** — sign-off was previously the author reading their own plan |
| `/implement <KEY\|path>` | Execute the approved plan step by step with verification and failure logging | plan | code, ticked plan, report | working tree, plan | planning-standards, testing-standards, git-standards, ai-failure-log, human-in-the-loop | lint/test commands | **New** — formalises what was conversational |
| `/self-review [--agent] [base]` | Hygiene + engineering review of the diff against standards; coverage table | diff | findings, coverage, verification results | nothing (offers fixes) | review-standards, security-review, performance-review, testing-standards, ai-failure-log | `code-reviewer` agent (optional) | Xiangqi `/review-pr` + server `pre-review-checklist`, merged |
| `/perf-check [base]` | Performance pass with impact estimates | diff | findings | nothing | performance-review | — | Xiangqi `/perf-check`, generic checklist |
| `/create-pr [KEY] [title]` | PR body contract + AI table → push → create → tracker → metadata → docs sync | branch | PR URL, report | GitHub, tracker (status), Confluence (optional) | jira-tracker, github-pr, git-standards, ai-contribution-scoring, human-in-the-loop | `gh` (required), Jira MCP (optional), Confluence MCP (optional) | Xiangqi `/create-pr`, all facts moved to config, confirm step added |
| `/pr-comments <PR>` | Implement review threads one by one; resolve on approval | PR | code changes, resolved threads | working tree, GitHub threads | github-pr, review-standards, git-standards | `gh` GraphQL | Xiangqi `/pr-review-comments`, trimmed |

## Side commands

| Command | Purpose | Origin |
| --- | --- | --- |
| `/ai-contribution [KEY]` | Print the AI table for the thread without a PR | Xiangqi `/ai-contribution` |
| `/log-failure [KEY] <text>` | Append an AI failure entry to the plan and (if general) the repo log | **New** — makes the ~20 % logging rate a two-second action |
| `/release-notes <version> [range]` | Draft notes from fixVersion + merged PRs, with unlinked lists | **New** — first release-stage support; drafts only |

## Considered and deferred

Proposed during analysis, not built — each would add a file without evidence it earns its place.
Add when a team asks for it twice.

| Candidate | Why deferred | If built, it should be |
| --- | --- | --- |
| `/clarify-requirements` | `/analyze-ticket` already emits numbered open questions; a separate command would duplicate it | part of analyze |
| `/security-review` | Claude Code ships one; the shared `security-review` skill runs inside self-review | skill (done) |
| `/test-strategy`, `/missing-tests` | Covered by the plan's test-strategy section and self-review's coverage table | skill (done) |
| `/release-readiness`, `/deployment-checklist`, `/migration-review` | Release process is outside Claude in the reference project; no observed practice to generalise | command, once a team has a written release checklist to encode |
| `/post-release-validation` | Needs monitoring access (APM, error tracker) that varies per team | agent with MCP access |
| `/tech-debt-scan`, `/dead-code`, `/dependency-audit`, `/refactor-candidates` | Maintenance commands are cheap to run ad hoc; making them commands risks noise reports nobody acts on | skill (`maintenance-review`) invoked on request |
| `/write-tests` | Test policy differs per repo; `testing-standards` + `/implement` cover it | skill (done) |
| `/run-review` | Not found; the name in the brief maps to `/self-review` | — |

## Project-specific commands that stay in their repos

`/request-translations`, `/apply-translations`, `/add-theme` and the `theme-creator` /
`board-creator` agents encode one product's locale files, translator and visual system. They
are good examples of *when to write a project command*: a multi-step, repeated, error-prone
task with repo-specific facts. The shared repo documents the pattern, not the commands.
