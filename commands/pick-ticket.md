---
description: List your ready-to-start tickets from the tracker, let you choose one, and hand off to /start-ticket. Never auto-picks.
---

Load `workflow-config`, `jira-tracker`, `human-in-the-loop`.

Requires `tracker.type = jira`. If not configured, say so and suggest `/setup-workflow`; if the
user just wants to start on something, point them to `/start-ticket <KEY>` or
`/analyze-ticket` with pasted text.

## Steps

1. **My queue** — run "My ready queue" from `jira-tracker` (active release, query, table).
   Extract with `jq` as described there; never read the raw dump.

2. **Ask** which ticket to work on. Wait. Never pick for the user. If the user wants several
   at once, point them to `/parallel-tickets`.

3. **Hand off** to `/start-ticket <KEY>`.

This command reads the tracker only; it never writes to it and never writes code.
