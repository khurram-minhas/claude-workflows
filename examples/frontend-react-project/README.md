# Example: multi-engineer React repository

The fullest configuration: Jira with story points and transitions, a reviewer pool, an
8-row AI contribution table (no Unit Tests row because the repo writes tests only on request),
a Confluence doc sync, and Figma enabled. This mirrors the reference project the shared
workflow was extracted from, with identifiers replaced.

What each part enables:

- `tracker.transitions` → `/start-ticket` and `/create-pr` move the ticket; the board is kept
  in sync by the commands rather than by dragging cards.
- `tracker.fields.plannedStoryPoints` → `/start-ticket` asks a human for the estimate and
  writes it; `/create-pr` reads it into the PR body.
- `github.reviewerPool` + `labels` → `/create-pr` requests the other pool members and labels
  the PR.
- `docsSync` → plans and command docs are mirrored to Confluence on PR creation.
- `failureLog.areas` → stack-specific buckets in `.claude/ai-failures.md`.

Pair with `templates/coding-standards/frontend-react.md` as `.claude/coding-standards.md`.
