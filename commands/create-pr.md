---
description: Open the pull request for the current branch — inspect the real diff, build the fixed PR body with the AI Contribution table, push, create via gh, then update the tracker (Code Review), assignee, labels and reviewers from config. Confirms before creating.
argument-hint: [TICKET] [title]
---

Load `workflow-config`, `jira-tracker`, `github-pr`, `git-standards`,
`ai-contribution-scoring`, `human-in-the-loop`.

## Steps

1. **Ticket** — from `$ARGUMENTS`, else from the branch name. None and no tracker → proceed
   without a ticket link; none *with* a tracker → ask.

2. **Inspect the changes** — `git fetch origin <base>`; `git diff origin/<base>...HEAD`;
   `git log origin/<base>..HEAD --oneline`. Refuse to write summary bullets the diff does not
   support. Uncommitted changes → stop and say so.

3. **Tracker read (one call)** — `getJiraIssue` with `fields: ["status", "issuetype",
   "<fields.plannedStoryPoints>"]` (only configured fields). Serves the transition gate, the bug
   label, and the Planned SP line.

4. **AI contribution table** — per `ai-contribution-scoring`, from this conversation + diff.

5. **Compose** — title from `git.prTitlePattern`; body per the `github-pr` contract (2–5
   bullets, table, SP lines, footer). If the repo has a PR template, follow its section order.

6. **Confirm** — if `github.confirmBeforeCreate` (default true), show title and body and wait
   for a yes. Edits requested here are applied before creating.

7. **Push and create** — push with `-u` if needed; `gh pr create --base <git.baseBranch>
   --title ... --body ...`. Print the URL.

8. **Tracker transition** — if `tracker.transitions.codeReview` is configured and the status is
   not already at or beyond Code Review, `transitionJiraIssue`. Never backwards. If
   `aiContribution.writeToTracker` is true and `fields.aiContribution` is configured, write the
   score shown in the table. Failures reported next to the PR link.

9. **PR metadata** — per `github-pr`: assignee (if `assignSelf`), labels (`readyForReview`;
   `bug` when the issue type is Bug), reviewers per `reviewerPolicy`. Read state first; report
   what was added versus already present versus failed.

10. **Docs sync (optional)** — if `docsSync.type` is `confluence` and the diff touches any of
    `docsSync.paths`, push those files to their mapped pages using `docsSync.map` — following
    the title and parent conventions recorded in that map file —
    (create the page under the mapped parent and add the map entry when the key is new; include
    the updated map in the branch). List created versus updated pages. Skip silently when not
    configured.

11. **Report** — PR URL, the AI table, tracker transition result, metadata result, docs sync
    result, and anything that failed or was skipped with the reason.

This command never merges. Story points are read, never estimated. Actual SP is never written.
