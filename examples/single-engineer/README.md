# Example: single engineer

The lightest setup that still gives the three things that matter most for one person:
grounded analysis (`/analyze-ticket` reads the ticket and the code), a written plan for
anything non-trivial (`/plan` — reviewed by `/plan-review` since there is no colleague to do
it), and a self-review before the PR (`/self-review`).

Deliberately absent: transitions (the board is small enough to drag), story points, reviewer
pool, docs sync. `/start-ticket` still shows the ticket and cuts the branch; it simply reports
"transition skipped — not configured".

Files to create with `/setup-workflow solo`: `.claude/workflow.json`, `.claude/CLAUDE.md`,
`.claude/coding-standards.md` (from the closest template), `.claude/plans/_template.md`,
`.claude/ai-failures.md`.
