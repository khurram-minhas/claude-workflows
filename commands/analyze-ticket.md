---
description: Technical analysis of a ticket against the real codebase — scope, ambiguities, affected files, architecture impact per layer, risks, dependencies and related work, ordered steps. Analysis only, no code.
argument-hint: <TICKET | pasted requirement>
---

Load `workflow-config`, `jira-tracker`, `figma-design` (if enabled), `ai-failure-log`,
`security-review`, `performance-review`.

## Inputs

`$ARGUMENTS` is a ticket key or pasted requirement text. With a tracker, fetch the ticket
(`fields: summary, description, issuetype, priority, fixVersions, issuelinks, labels`). Read
`project.codingStandards`, `project.additionalContext`, and the repo-wide failure log.

## Ground the analysis before writing it

Read the actual files that the requirement touches — search by feature name, route, event
name, model name. Cite `file:line`. Do not analyse from memory of "how such apps usually work".
If a Figma link is present and `figma.enabled`, pull the design context.

## Produce

1. **Requirement restated** — acceptance criteria as a checklist, quoting the ticket. Mark
   anything ambiguous with a numbered **open question** (do not resolve it yourself).
2. **Related work** — linked issues from the tracker; recent commits/PRs in the same area
   (`git log --oneline -20 -- <paths>`); anything in the failure log for this area.
3. **Files to create or modify** — exact paths, one line each on what changes.
4. **Architecture impact** — one row per layer that applies, drawn from the stack in
   `project.stackSummary` and the standards file. Typical layers: data model / migrations, API
   or event contracts, state management, async or background work, real-time events, i18n,
   styling and design tokens, configuration and feature flags, analytics. Drop rows that do
   not apply.
5. **Risks** — correctness, concurrency, performance (`performance-review` items that apply),
   security (`security-review` trigger check — say explicitly if it is triggered), quality-gate
   or lint rules likely to bite.
6. **Test strategy sketch** — what will prove it works; regression test if it is a bug.
7. **Rollout / rollback** — only if a flag, migration, or contract change is involved.
8. **Ordered implementation steps** — smallest reviewable order.
9. **Size check** — is this one PR? If not, propose the split.

Do **not** write code. End by offering `/plan <KEY>` to turn this into a plan file, or, if the
change is trivial and `plans.requiredFor` allows, to proceed straight to implementation with the
user's explicit ok.
