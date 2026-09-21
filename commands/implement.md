---
description: Execute an approved plan step by step — read the plan and the standards, implement each step, run the project's lint/test commands, tick the plan's checkboxes, log AI failures as they happen, and stop at scope boundaries.
argument-hint: <TICKET | plan file path>
---

Load `workflow-config`, `planning-standards`, `testing-standards`, `git-standards`,
`ai-failure-log`, `human-in-the-loop`.

## Preconditions

- Plan exists and has `**Approved:**` filled in. If not: stop, point to `/plan` /
  `/plan-review`. If `plans.requiredFor` is `never` or the user explicitly skipped planning for
  a trivial change, proceed from the agreed description instead and say so.
- Current branch matches the ticket (`git-standards`); otherwise stop.
- Read `project.codingStandards`, `project.additionalContext`, and the repo-wide failure log.
  Quote any failure-log entry that overlaps this work before starting.

## Loop — per plan step

1. Read the files the step names **before** editing them; match their patterns exactly.
2. Implement the step. No new abstractions, dependencies, or files beyond what the plan lists
   without asking first.
3. Verify: `commands.lint` (and `format`/`typecheck` if configured) on changed files;
   `commands.testSingleFile` for the tests this step touches. Show the output; fix before
   moving on.
4. Tick the step in the plan (`- [x]`). If the step's reality differed from the plan, update
   the plan text — the plan must describe what was actually done.
5. If you were corrected or a verification failed for a foreseeable reason, add an
   `## AI failure points` entry in the plan now.

## Scope boundaries

Anything outside the plan's "Files" table or inside "Out of scope" is a **question**, not an
action. Ask, offering: add to the plan (and do it), defer to a follow-up ticket, or skip.

## Finish

- Run `commands.test` (full, if configured and `testing.notes` permits) once.
- Update the "Done when" checkboxes only for criteria you actually verified; leave the rest
  unchecked and list them.
- Report: steps done, verification commands run with results, deviations from the plan,
  failure-log entries added, unchecked criteria.
- Suggest `/self-review`.

Commit only if the user asks (`git-standards`). This command never creates a PR.
