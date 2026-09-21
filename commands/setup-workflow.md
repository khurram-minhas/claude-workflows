---
description: Bootstrap the shared workflow in a repository — detect the stack, verify tracker IDs, write .claude/workflow.json, and scaffold the plan template, failure log, coding standards and PR template. Run once per repository.
argument-hint: [solo|team]
---

Set up the arbisoft-workflows commands for this repository. Load `workflow-config` and
`jira-tracker` first.

Profile from `$ARGUMENTS`: `solo` (one engineer, minimum files) or `team` (default — full set).
Ask if neither is given.

**Never overwrite an existing file without showing the diff and getting a yes.**

## 1. Detect what is already here

```bash
ls .claude/ 2>/dev/null; cat .claude/workflow.json 2>/dev/null
ls CLAUDE.md .claude/CLAUDE.md .github/PULL_REQUEST_TEMPLATE.md .github/pull_request_template.md 2>/dev/null
git remote -v | head -2; git branch -r | sed 's|origin/||' | head -20
ls package.json pyproject.toml requirements.txt go.mod Cargo.toml pom.xml 2>/dev/null
```

From the manifests, propose `project.stackSummary` and the `commands.*` entries (lint, test,
typecheck, build). Show what you inferred and ask for corrections. From remote branches, propose
`git.baseBranch` and the branch naming pattern actually in use.

## 2. Tracker

Ask which tracker: Jira, GitHub Issues, or none.

For Jira, if the Atlassian MCP is available:

1. `getAccessibleAtlassianResources` → confirm `cloudId`.
2. Ask for the project key; verify with `getVisibleJiraProjects`.
3. Ask for one representative ticket key. Call `getTransitionsForJiraIssue` on it and show the
   list; the user picks which ids mean "In Progress" and "Code Review" (or says the team does
   not transition tickets from commands).
4. `getJiraIssueTypeMetaWithFields` for a Story → find the Story Points field id (and Actual
   Points / AI Contribution fields if the team tracks them). Show the candidates; the user
   confirms. **Write only confirmed ids.**
5. Ask whether `/pick-ticket` should filter to the active release.

If the MCP is not available, write `tracker.type` and `projectKey` only and leave ids `null`
with a note that `/setup-workflow` can be re-run once the MCP is connected.

## 3. GitHub

`gh auth status`. Ask about: label for ready-to-review PRs, bug label, reviewer pool (GitHub
logins), whether to assign the author. `solo` profile: reviewer pool empty, labels optional.

## 4. Policies

Ask, offering the defaults: testing policy, regression-test rule, plans required for
(`all` / `non-trivial` / `never`), AI contribution rows (show the default table; drop rows the
repo never does), whether the AI % should be written back to the tracker field (default no).

## 5. Write files

- `.claude/workflow.json` — only keys that differ from the defaults, plus `$schema` pointing
  at `config/workflow.schema.json` in this plugin. Print it in full.
- Add `.claude/workflow.local.json` to `.gitignore`.
- `plans.template` from `templates/plan-template.md` (skip for `solo` if `plans.requiredFor`
  is `never`).
- `failureLog.path` from `templates/ai-failures.md` with the configured areas as headings.
- `project.codingStandards` from the closest template in `templates/coding-standards/`
  (`frontend-react.md`, `backend-python.md`, `generic.md`) — ask which, and tell the user the
  stack-specific sections are a starting point to edit, not a standard.
- PR template from `templates/PULL_REQUEST_TEMPLATE.md` with the configured weights, if the repo
  has none.
- Append the "Workflow" section from `templates/CLAUDE.md.template` to the project's
  `CLAUDE.md` (create it from the template if absent), pointing at the config and standards.

## 6. Report

One block: files created / updated / skipped, config keys still `null` and what each disables,
and the recommended first command (`/pick-ticket` if the tracker is configured, otherwise
`/analyze-ticket <paste>`).
