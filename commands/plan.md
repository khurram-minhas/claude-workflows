---
description: Create the plan file for a ticket from the project template, filled from a fresh analysis, and ask for human sign-off before any code is written.
argument-hint: <TICKET> <short description>
---

Load `workflow-config`, `planning-standards`, `testing-standards`, `human-in-the-loop`.

## Steps

1. **Gate** — with a tracker configured, the ticket should already be In Progress with Planned SP
   set (that is `/start-ticket`'s job). If not, say so and run `/start-ticket <KEY>` first. Do
   not set SP or transition from here.

2. **Is a plan needed?** Apply `plans.requiredFor`. If the change is trivial under the
   `planning-standards` definition, say so, and offer to skip to implementation with the user's
   ok instead of producing a ceremonial plan.

3. **Analyse** — run `/analyze-ticket <KEY>` unless an analysis for this ticket already exists
   in this conversation.

4. **Create the file** — `<plans.dir>/<KEY>-<slug>.md` from `plans.template`. If the template is
   missing, use `templates/plan-template.md` from this plugin and say so. Fill every section
   per `planning-standards`; delete rows that do not apply rather than writing "N/A". Quote the
   ticket; cite file:line; include design links if any; leave `**Approved:**` empty.

5. **Present** the plan and ask for sign-off. Offer `/plan-review` for an independent challenge
   before approval on anything non-trivial. Record approval in the header when it is given
   (`**Approved:** <name>, <date>`).

No implementation code is written by this command. Implementation starts with
`/implement <KEY>` once the plan is approved.
