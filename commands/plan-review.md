---
description: Challenge a plan before it is approved — coverage of acceptance criteria, unverified assumptions, root-cause proof, risk, test strategy, scope — using an independent reviewer context. Produces proposed edits to the plan, never code.
argument-hint: <TICKET | plan file path>
---

Load `workflow-config`, `planning-standards`, `review-standards`.

## Steps

1. **Locate the plan** — `$ARGUMENTS` is a ticket key (→ `<plans.dir>/<KEY>-*.md`) or a path.
   Read it and the ticket text it quotes.

2. **Independent review** — dispatch the `plan-reviewer` agent with: the plan path, the
   ticket text, `project.stackSummary`, and the path to `project.codingStandards`. Its value is
   a fresh context that has not been talking itself into the plan for an hour. If agents are
   unavailable, do the review inline but say so — an inline self-check is weaker.

3. **Verify the findings** — per `review-standards` → "Receiving review": check each claim
   against the codebase before presenting it. Drop findings that do not hold up and say why.

4. **Present** — findings grouped by severity, each citing the plan section and (where relevant)
   the code that contradicts it, with a concrete proposed edit to the plan.

5. **Apply what the human accepts** — edit the plan file for accepted findings only. Then ask
   for sign-off and record `**Approved:**` in the header if given.

This command never writes implementation code and never changes the ticket.
