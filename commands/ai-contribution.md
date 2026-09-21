---
description: Print the AI Contribution Checklist for the work done on a ticket in this conversation, without creating or touching a PR or the tracker.
argument-hint: [TICKET]
---

Load `workflow-config`, `ai-contribution-scoring`.

1. Ticket from `$ARGUMENTS`, else the branch name, else "current work".
2. Review this conversation (and `git log` / `git diff` against the base if commits exist)
   for everything done toward it.
3. Score per `ai-contribution-scoring` with the configured weights. Blank, never zero, for
   activities that have not happened yet.
4. Print the table and the score, followed by the honesty statements from the skill.
5. Remind that Planned SP and Actual SP are tracked on the ticket by humans and are not
   computed here.

Reports only; edits nothing.
