---
description: Open a ticket for development — show it, settle the story points with a human, move it to In Progress, and cut the branch from an up-to-date base. Run before /plan.
argument-hint: <TICKET> [story-points]
---

Load `workflow-config`, `jira-tracker`, `git-standards`, `human-in-the-loop`.

This is the **only** command that moves a ticket to In Progress or writes story points, so the
board reflects when work actually starts.

## Steps

1. **Resolve the ticket** — from `$ARGUMENTS`, else from the branch name. If neither, ask.

2. **Fetch and show it** (`jira-tracker` → "Presenting a ticket"): link, summary, type,
   priority, fix version, status, and the description quoted. If the project's `CLAUDE.md` says
   tickets span several repos, confirm this one belongs here.

3. **Story points** — only if `tracker.fields.plannedStoryPoints` is configured:
   - Already set → state it, leave it.
   - Empty and a value was passed → use it.
   - Empty and nothing passed → ask, offering `tracker.storyPointScale` values with a one-line
     reason each grounded in the description, plus "decide after investigating". Never choose
     yourself.
   - Write with `editJiraIssue`; confirm the value written.
   - Never touch `tracker.fields.actualStoryPoints`.

4. **Transition** — only if `tracker.transitions.inProgress` is configured. Skip (and say so) if
   the status is already In Progress or in `statuses.atOrBeyondCodeReview`. Never move backwards.

5. **Branch** — per `git-standards`: fetch the base, create `<git.branchPattern>` from
   `origin/<git.baseBranch>`. Dirty tree or existing branch → stop and report.

6. **Report** — one block: ticket link, Planned SP (set now / already set / not tracked),
   transition (applied / skipped and why / not configured), branch checked out. Any failed
   tracker call is listed as a failure here, not hidden.

7. **Hand off** — suggest `/plan <KEY> <short description>` (or `/analyze-ticket <KEY>` if
   `plans.requiredFor` is `never`). This command never writes code.
