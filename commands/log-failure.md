---
description: Record an AI mistake in the fixed format — in the current ticket's plan and, if it generalises, in the repo-wide AI failures log. Two-second action so corrections are actually written down.
argument-hint: [TICKET] <what went wrong → what was correct>
---

Load `workflow-config`, `ai-failure-log`.

1. Ticket from `$ARGUMENTS` or the branch. Entry text from `$ARGUMENTS`; if it is missing,
   draft it from the most recent correction in this conversation and show it for approval.
2. Format exactly `- [<TICKET>] <what the AI did wrong> → <correct approach>`. Make it
   specific (file, function, assumption, the check that would have caught it).
3. Append to the plan's `## AI failure points` section if a plan exists.
4. Ask: "Does this apply beyond this ticket?" If yes, append under the matching area heading
   in `failureLog.path` (create the file from `templates/ai-failures.md` if absent).
5. Show what was written and where.
