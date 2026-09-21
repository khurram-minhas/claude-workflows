---
description: Review the branch's diff before a human sees it — hygiene pass (scope vs plan, dead code, debug leftovers, lint), then engineering pass (correctness, error handling, patterns, security, performance, tests) against the project's standards. Optionally dispatches an independent reviewer.
argument-hint: [--agent] [base-ref]
---

Load `workflow-config`, `review-standards`, `security-review`, `performance-review`,
`testing-standards`, `ai-failure-log`.

## Scope

Diff = `git diff origin/<git.baseBranch>...HEAD` plus uncommitted changes (`git diff`,
`git status --short`). A base ref in `$ARGUMENTS` overrides. Read the plan for this ticket if
one exists — its "Files" table and "Done when" list are the reference for scope and coverage.

## Steps

1. **Pass 1 — hygiene** (`review-standards`). Run `commands.lint` / `format` / `typecheck` on
   changed files and show the output.
2. **Pass 2 — engineering judgement** (`review-standards`), applying `security-review` and
   `performance-review` where triggered and the project's own checklist from
   `project.codingStandards`.
3. **Requirement coverage** — map each "Done when" criterion to evidence in the diff or to a
   manual check that was performed. Unverified criteria are listed, not assumed.
4. **Tests** — run `commands.test` if configured and safe per `testing.notes`. Confirm the
   regression test exists for bug fixes and fails without the fix (describe how you checked).
5. **Independent pass** — if `--agent` is given or `review.useIndependentReviewer` is true,
   dispatch the `code-reviewer` agent on the same diff and merge its verified findings.
6. **Report** — findings by severity with file:line and a fix for each; then the coverage
   table; then the verification commands run and their results.
7. **Offer to fix** the 🔴 items now. Re-run the relevant verification after each fix.
8. **Failure log** — ask whether any of the plan's AI failure points should be promoted to the
   repo-wide log; do it if yes.

End by suggesting `/perf-check` for hot-path changes and `/create-pr` when clean.
