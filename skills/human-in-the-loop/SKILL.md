---
name: human-in-the-loop
description: The division of responsibility between the AI and the engineer that every shared command enforces — what the AI may do unprompted, which decisions always go to a human, and the explicit gates (ticket pick, story points, plan sign-off, PR creation, thread resolution, merge). Load when a command is about to take an outward-facing or irreversible action.
---

# Human in the loop

## The AI assists with

Context gathering, ticket analysis, drafting plans, implementation against an approved plan,
tests, documentation, PR preparation, review assistance, release-note drafts.

## Humans remain responsible for

Requirements and their interpretation, architecture, estimates, plan approval, code review and
merge, security-sensitive decisions, product decisions, final quality.

## Gates (the command stops and waits)

| Gate | Command | What the human decides |
| --- | --- | --- |
| Which ticket | `/pick-ticket` | Never auto-picked |
| Story points | `/start-ticket` | Never invented; options shown against the real ticket text |
| Plan approval | `/plan`, `/plan-review` | Approval recorded in the plan header before `/implement` |
| Deviating from the plan | `/implement` | Any step outside the plan's scope is a question, not an action |
| PR creation | `/create-pr` | Title/body shown first when `github.confirmBeforeCreate` is true |
| Resolving a review thread | `/pr-comments` | Only after the change is shown and approved |
| Merge / release | — | No shared command merges or deploys |

## Escalate, don't decide

Product trade-offs, data retention, permissions, anything with money or legal weight, and any
"the ticket and the design disagree" situation are presented as options with a recommendation.
The AI recommends; the human chooses.

## Honesty rules

- Report failures next to successes; never fold an error into a success message.
- "Done" means verified: the command that proves it was run and its output seen.
- Say what was skipped and why (missing config, unavailable MCP, denied permission).
- Estimates (AI %, effort) are labelled as estimates.
