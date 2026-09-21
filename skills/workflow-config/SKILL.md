---
name: workflow-config
description: How every shared command loads and resolves the project's workflow configuration (.claude/workflow.json), what to do when a key is missing, and how to never guess a project fact. Load at the start of any arbisoft-workflows command.
---

# Workflow configuration

Shared commands contain no project facts. Everything that varies per project — tracker keys,
field IDs, branch names, labels, reviewers, lint/test commands — comes from configuration.

## Resolution order (later wins)

1. Built-in defaults: `config/workflow.defaults.json` in this plugin
   (`${CLAUDE_PLUGIN_ROOT}/config/workflow.defaults.json` when installed as a plugin).
2. Any file named by `extends` in the project file, resolved recursively (a team-wide file
   shared by several repositories usually lives here).
3. Project file: `.claude/workflow.json` — committed, owned by the team.
4. Personal file: `.claude/workflow.local.json` — git-ignored, owned by the engineer.
5. Explicit arguments passed to the command (`$ARGUMENTS`).

Merge is a deep merge on objects; arrays replace rather than concatenate.

## How to load it

At the start of a command:

```bash
cat .claude/workflow.json 2>/dev/null; cat .claude/workflow.local.json 2>/dev/null
```

Read the defaults file once as well. Keep the merged result in your head for the rest of the
command; do not re-read it per step.

If `.claude/workflow.json` does not exist:

- Say so once, in one line.
- Suggest `/setup-workflow` for a guided setup.
- Continue in **no-tracker mode** where the command can still be useful (analysis, planning,
  review, PR creation without tracker updates). Stop only if the command's whole purpose is
  tracker-bound (`/pick-ticket`, `/start-ticket`).

## Rules

- **A `null` key means "not configured", not "guess".** Skip the behaviour that needs it and say
  which step was skipped and why. Example: `tracker.transitions.inProgress` is null → do not
  transition the ticket; report "transition skipped — not configured".
- **Never invent IDs.** Custom field IDs and transition IDs are looked up with
  `getJiraIssueTypeMetaWithFields` / `getTransitionsForJiraIssue` and written into config by
  `/setup-workflow`. If a command finds them missing, ask — it may write them back to
  `.claude/workflow.json` only if the user agrees.
- **Read the project's standards file** (`project.codingStandards`) whenever a command analyses,
  plans, implements or reviews code. The shared commands describe *what* to check; that file
  says *how it is done in this codebase*.
- **Quote `project.stackSummary`** into any prompt-like step so the analysis is grounded in the
  real stack rather than a generic one.
- Placeholders in patterns: `<TICKET>` (e.g. ABC-123), `<slug>` (kebab-case, ≤ 5 words, from the
  ticket summary), `<name>` (git user's first name, lowercase), `<summary>` (imperative, ≤ 70
  chars).
- Ticket keys are matched with `<projectKey>-\d+`. When `tracker.projectKey` is null, accept any
  `[A-Z][A-Z0-9]+-\d+` from arguments or the branch name, but never derive a key from nothing.
