---
name: jira-tracker
description: Jira integration rules for shared commands — which Atlassian MCP tools to call for fetching, searching, transitioning and editing issues, how to keep large results out of context, the never-move-backwards and never-invent-story-points rules, and the paste fallback when the MCP is unavailable.
---

# Jira tracker integration

Applies when `tracker.type` is `jira`. All calls take `cloudId: <tracker.cloudId>`.

## Tool map

MCP tool names vary by how the Atlassian server was registered (`mcp__atlassian-rovo-mcp__*`,
`mcp__claude_ai_Atlassian_Rovo__*`, …). Use whichever prefix is present; the suffixes are stable:

| Need | Tool | Notes |
| --- | --- | --- |
| One issue | `getJiraIssue` | Always pass `fields: [...]` — the default payload is large. Use `responseContentFormat: "markdown"` for descriptions you will show. |
| Search | `searchJiraIssuesUsingJql` | `maxResults ≤ 50`, `fields` limited to what you display. |
| Transitions available | `getTransitionsForJiraIssue` | Use to *verify* an id, never to pick one silently. |
| Move status | `transitionJiraIssue` | `transition: { id: "<id>" }` |
| Set a field | `editJiraIssue` | `fields: { "<customfield_id>": <value> }` |
| Comment | `addCommentToJiraIssue` | Only when a command says it comments. |
| Field ids for setup | `getJiraIssueTypeMetaWithFields` | Used by `/setup-workflow`. |

## Keeping results out of context

Search results are usually written to a file by the tool. Extract only what you need:

```bash
jq -r '.issues.nodes[] | "\(.key)\t\(.fields.priority.name // "-")\t\(.fields.status.name)\t\(.fields.summary)"' <saved-file>
```

Never read a raw Jira JSON dump into context.

## Presenting a ticket

Before asking the user anything about a ticket, show:

- the clickable link: `<tracker.browseUrl><TICKET>`
- summary, issue type, priority, fix version(s), current status
- the **description as written** — repro steps, expected/actual, acceptance criteria. Quote the
  substance; do not replace it with a paraphrase. Estimates and plans must be made against the
  real requirement.

## Rules

- **Never move a ticket backwards.** If its status is already the target or is in
  `tracker.statuses.atOrBeyondCodeReview`, skip the transition and say so.
- **Never invent Story Points.** If `fields.plannedStoryPoints` is configured and empty, ask
  (offer values from `tracker.storyPointScale` plus "decide after investigating"), grounding each
  option in what the ticket says. Write the human's answer verbatim.
- **Never write `fields.actualStoryPoints`.** Humans only.
- **Write `fields.aiContribution` only if `aiContribution.writeToTracker` is true**, and only
  the value shown to the user in the PR table.
- **A failed call is reported as a failure**, next to whatever did succeed. Never fold it into a
  success message. A Jira blip must not stop git work (branch creation, PR creation), but it
  must be visible in the final report.
- **Cross-repo tickets:** if the project's `CLAUDE.md` says tickets are shared across repos,
  confirm the ticket's description points at *this* repo before starting it.

## Fallback when the MCP is not available

Say in one line that the Atlassian MCP is not connected (`/mcp` in an interactive session, or
the claude.ai connector settings), then offer the paste fallback: the user pastes the ticket
text (summary + description + status). Continue the command with that text. Skip every write
step (transitions, field edits, comments) and list them at the end as "to do by hand".
