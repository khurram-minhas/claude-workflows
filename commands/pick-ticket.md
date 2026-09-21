---
description: List your ready-to-start tickets from the tracker, let you choose one, and hand off to /start-ticket. Never auto-picks.
---

Load `workflow-config`, `jira-tracker`, `human-in-the-loop`.

Requires `tracker.type = jira`. If not configured, say so and suggest `/setup-workflow`; if the
user just wants to start on something, point them to `/start-ticket <KEY>` or
`/analyze-ticket` with pasted text.

## Steps

1. **Active release (optional)** — if `tracker.pickFilter.onlyActiveRelease` is true, search
   `project = <projectKey> AND fixVersion in unreleasedVersions("<projectKey>")` with
   `fields: ["fixVersions"]`, collect unreleased versions, pick the earliest `releaseDate`. If
   none, ask which release to use rather than guessing.

2. **My queue** — JQL:
   ```
   project = <projectKey> AND assignee = currentUser() AND status = "<statuses.ready>"
   [AND fixVersion = "<active release>"] [AND <pickFilter.extraJql>]
   ORDER BY priority DESC, updated DESC
   ```
   `maxResults: 25`, `fields: ["summary", "status", "priority", "issuetype", "fixVersions"]`.
   Extract with `jq` as described in `jira-tracker`; never read the raw dump.

3. **Present** a table — key, type, priority, summary, fix version — and state the filter that
   was applied. If empty, say so and offer to broaden (other statuses, unassigned, other
   release) on request.

4. **Ask** which ticket to work on. Wait. Never pick for the user.

5. **Hand off** to `/start-ticket <KEY>`.

This command reads the tracker only; it never writes to it and never writes code.
