---
name: planning-standards
description: What a good implementation plan contains, when one is required, the sign-off gate, and the rubric used to review or challenge a plan (coverage, assumptions, risk, test strategy, rollout, scope). Used by /plan, /plan-review, /implement and the plan-reviewer agent.
---

# Planning standards

## When a plan is required

`plans.requiredFor` decides: `all`, `non-trivial` (default), or `never`. "Non-trivial" means any
of: more than ~3 files, a data-model or API contract change, a new user-facing flow, anything
touching auth/payments/data deletion, or a bug whose cause is not yet known. A one-line fix with
an obvious cause does not need a plan — say so and move on.

## Where plans live

`<plans.dir>/<TICKET>-<slug>.md`, created from `plans.template`. One plan per ticket. The plan
is committed with the code so the reasoning survives the branch.

## Required sections (the template enforces them)

1. **Header** — ticket link, type, priority, fix version, Planned SP (read, never estimated),
   branch, date, design links if any.
2. **Ticket** — the requirement quoted, not paraphrased.
3. **Goal** — one paragraph.
4. **Root cause** (bugs) or **Approach** (features) — grounded in file:line references that were
   actually read.
5. **Files to create or modify** — table of path / action / note.
6. **Architecture impact** — per layer that applies (data model, API contracts, state,
   async/side effects, real-time events, i18n, styling, config/feature flags). Drop rows that
   don't apply; do not leave "N/A" noise.
7. **Edge cases and risks** — including concurrency and failure modes.
8. **Test strategy** — what is verified and how; regression test line for bug fixes (see
   `testing-standards`).
9. **Rollout / rollback** — only when the change is behind a flag, has a migration, or changes a
   contract. Otherwise one line: "Plain deploy; revert = revert the PR."
10. **Out of scope** — explicit, so scope creep must be argued for.
11. **Done when** — acceptance criteria as checkboxes.
12. **AI failure points** — filled during and after the work (`ai-failure-log`).

## The sign-off gate

A plan is presented to a human and **explicitly approved** before implementation starts. Record
the approval in the plan header (`**Approved:** <name>, <date>`). `/implement` refuses to start
without it. Changing the plan mid-implementation is fine — update the file and say what changed.

## Plan review rubric

Used by `/plan-review` and the `plan-reviewer` agent. For each item, cite the plan line and,
where relevant, the code that contradicts it.

| # | Question | Typical failure |
| --- | --- | --- |
| 1 | Does every acceptance criterion in the ticket map to a step or a "Done when" line? | Criteria silently dropped |
| 2 | Are the file references real and current? | Plan written against a stale mental model |
| 3 | What is assumed but not verified? | "The API already returns X" |
| 4 | Is the root cause proven (bugs) or is it the first plausible story? | Fix that hides the symptom |
| 5 | What breaks if this is half-deployed, retried, or raced? | No thought about concurrency |
| 6 | Is the test strategy proportional to risk? | "Manual testing" for a data migration |
| 7 | Is anything security-sensitive touched (auth, PII, money, deletion)? | Not flagged for a human decision |
| 8 | Is the scope one PR's worth? | Should be split |
| 9 | Is "Out of scope" honest, or is it hiding required work? | Follow-up ticket that never comes |
| 10 | Does it introduce a new abstraction the codebase does not have? | "No new abstractions unless requested" |

Severity uses `review.severityScale` (🔴 / 🟡 / 🟢). A plan review produces **proposed edits to
the plan**, never code.
