# Configuration Guide

Shared commands contain no project facts. Everything that varies is configuration, resolved in
layers:

```
Shared defaults        config/workflow.defaults.json          (in the plugin; never edited per project)
        ↓
Team configuration     e.g. team-config/workflow.team.json     (one file for all repos of a team; via "extends")
        ↓
Project configuration  .claude/workflow.json                   (committed; owned by the team)
        ↓
Engineer configuration .claude/workflow.local.json             (git-ignored; personal overrides)
        ↓
Command arguments      $ARGUMENTS                              (per invocation)
```

Deep-merge on objects, replace on arrays, later wins. A `null` value means *not configured*:
the command skips that behaviour and says so. It never guesses.

## The file

Schema: [`config/workflow.schema.json`](../config/workflow.schema.json). Defaults:
[`config/workflow.defaults.json`](../config/workflow.defaults.json). Create it with
`/setup-workflow`, which verifies tracker ids live instead of copying them.

| Section | What it controls | Used by |
| --- | --- | --- |
| `project` | name, one-line stack summary, path to coding standards, extra context files | analyze, plan, implement, self-review, perf-check |
| `tracker` | Jira cloudId, project key, browse URL, custom field ids, status names, transition ids, pick filter, SP scale | pick, start, analyze, create-pr, release-notes |
| `git` | base branch, branch / PR title / commit title patterns | start, implement, create-pr, pr-comments |
| `github` | repo, self-assign, labels, reviewer pool and policy, confirm-before-create | create-pr, pr-comments |
| `commands` | lint, format, typecheck, test, single-file test, build | implement, self-review, perf-check, pr-comments |
| `testing` | policy, regression rule, dangerous-facts notes | plan, implement, self-review |
| `plans` | directory, template path, when required | plan, plan-review, implement |
| `review` | severity labels, independent reviewer by default | self-review, plan-review |
| `aiContribution` | enabled, weights, write-back to tracker | create-pr, ai-contribution |
| `failureLog` | path, area headings | analyze, implement, self-review, log-failure |
| `docsSync` | none / confluence, map path, watched globs | create-pr |
| `figma` | enabled | analyze |

## Typical variations

| Situation | Set |
| --- | --- |
| No tracker at all | `tracker.type: "none"` — analysis/plan/review/PR still work; ticket keys optional |
| Tracker but the board is dragged by hand | leave `tracker.transitions.*` null |
| No story points | leave `tracker.fields.plannedStoryPoints` null |
| Repo never writes unit tests | remove the `Unit Tests` row from `aiContribution.weights` and re-balance to 100 |
| Dangerous test suite | `commands.test: null`, set `commands.testSingleFile`, describe the danger in `testing.notes` |
| Several repos, one team | put tracker/github/weights in a team file; each repo `extends` it |
| Personal reviewer preferences, local lint wrappers | `.claude/workflow.local.json` |

## What must never go in the shared repo

Project names, Jira keys and custom-field ids, GitHub repos and logins, label names, Confluence
page ids, people. If a pull request to this repository contains any of these outside
`examples/`, it is wrong.
