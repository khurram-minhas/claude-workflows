# Arbisoft Claude Adoption Guide

A coaching guide for helping an Arbisoft team adopt AI-assisted engineering the way the
Xiangqi team did — as a written, configurable, human-gated workflow rather than a chat window.
It is a framework, not a mandate: every team starts where it is and takes the next step that
pays for itself.

Companion documents: [Xiangqi-AI-Journey.md](Xiangqi-AI-Journey.md) (the case study),
[Command-Catalog.md](Command-Catalog.md), [Skills-Catalog.md](Skills-Catalog.md),
[MCP-Integration-Guide.md](MCP-Integration-Guide.md), [Configuration-Guide.md](Configuration-Guide.md).

---

## 1. The principle everything rests on: human in the loop

| The AI assists with | The engineer remains responsible for |
| --- | --- |
| Context gathering, ticket analysis | Requirements and their interpretation |
| Drafting plans | Architecture and plan approval |
| Implementation against an approved plan | Estimates |
| Tests, documentation, PR preparation | Code review and merge |
| Review assistance, release-note drafts | Security-sensitive and product decisions, final quality |

The shared commands make this concrete with **gates** — points where the command stops and
waits: which ticket, the story-point estimate, plan sign-off, deviating from the plan, PR
creation, resolving a review thread. No shared command merges or deploys. Anything with
product, legal or money weight is presented as options with a recommendation.

Coach's framing: *the AI gets faster and the engineer gets more careful — that is the deal.*

---

## 2. Where Xiangqi started, and the direction it took

Start state, every team's default:

```
Jira requirement → copy/paste into Claude → ask for help → engineer does the rest by hand
```

Direction, reached in about three months:

```
Pick ticket → Analyze → Plan → Plan review → Implementation → Testing → Code review → PR
           → PR review cycle → Measurement → Continuous improvement
```

What made the difference was not the model; it was that each step became a **file** — an
instruction that survives context resets and people changes — and that Jira, GitHub and Figma
were reached through MCPs and the `gh` CLI instead of copy-paste. Context transfer, the thing
engineers spent most of their AI time on, was removed from the loop.

---

## 3. Two adoption models

### Model A — single engineer / small project

One person, one repository, no reviewer pool. The goal is *grounded analysis, a written plan,
and a self-review* — the three things that most reduce rework when there is no colleague to
catch mistakes.

```
Project context  →  CLAUDE.md  →  Coding standards  →  /analyze-ticket  →  /plan (+ /plan-review)
                 →  /implement  →  /self-review  →  /create-pr  →  human review (if any)  →  /ai-contribution
```

Minimum set: `/setup-workflow solo`, `/analyze-ticket`, `/plan`, `/plan-review`, `/implement`,
`/self-review`, `/create-pr`. Skills come with them. Configuration: see
`examples/single-engineer/`.

Explicitly *not* required: ticket transitions, story points, reviewer pools, docs sync,
Confluence, agents beyond the two shipped. `/plan-review` matters more here, not less —
the independent agent is the second pair of eyes the engineer does not otherwise have.

Basic metrics: the AI table on each PR and the plan file per ticket. Nothing else.

### Model B — multi-engineer team

Several people, one or more repositories, a shared board. The goal is *consistency where it
matters and freedom where it doesn't*.

Establish, in this order:

1. **Shared Claude instructions** — one `CLAUDE.md` shape per repo (template provided);
   architecture map, NEVER list, workflow table.
2. **Shared coding standards** — `.claude/coding-standards.md`, core rules + stack rules +
   "lessons from code review". Owned like code: PRs, reviews.
3. **Shared commands and skills** — install the plugin; do not fork commands per person.
4. **Common MCP integrations** — Jira at user scope for everyone; `gh` authenticated; Figma
   where the team has designs.
5. **Planning standard** — plans required for non-trivial work, sign-off recorded, plan review
   for anything large or risky.
6. **Review standard** — self-review before opening a PR; human review on GitHub; threads
   worked through `/pr-comments`.
7. **AI contribution tracking** — the PR table with agreed weights; Planned SP from a human;
   Actual SP by hand.
8. **Team-level metrics** — a delivery view (release → ticket → SP/AP → AI %) with no
   individual ranking.

Consistency vs freedom: the *artifacts* are standard (plan file, PR body, standards file,
config); *how* each engineer talks to Claude is not. One person may use `/implement` step by
step, another may implement conversationally and only use `/self-review` — both produce the
same plan and PR shape. Put the team's real facts in a `workflow.team.json` that every repo
`extends`, so nobody maintains a copy.

---

## 4. Adoption roadmap

Each phase is worth doing on its own. Skip nothing, but stop anywhere.

| Phase | Do | Done when |
| --- | --- | --- |
| **1 — Context** | `CLAUDE.md` from the template; `coding-standards.md` with the *true* rules of this codebase; `/setup-workflow` | A new engineer's first Claude session follows the house patterns without being told |
| **2 — Workflow** | `/analyze-ticket`, `/plan`, `/implement`, `/self-review` on real tickets for two weeks | Plans exist for non-trivial tickets; self-review findings appear before human review |
| **3 — Integration** | Jira MCP (reads first, then SP + transitions), `gh`, Figma where designs exist | No ticket text is pasted by hand; the board follows the commands |
| **4 — Automation** | `/create-pr` with the body contract, `/pr-comments`, project-specific commands for repeated multi-step tasks, agents where an independent context is the point | PR bodies are uniform; repeated chores have a command |
| **5 — Measurement** | AI table on every PR, Planned SP from `/start-ticket`, Actual SP by hand, a delivery view over releases | The team can say what was planned, delivered and how AI was involved, per release |
| **6 — Continuous improvement** | `ai-failures.md` read before work and written after; monthly look at which commands are used, which are not, what is still manual | Commands change because usage says so; failure entries stop repeating |

---

## 5. AI adoption maturity model

A way for a team to locate itself and pick the next capability. Levels describe capabilities,
not quality of people — a level-2 team with a good standards file beats a level-4 team with a
stale one.

| Level | Name | What is true | Typical next step |
| --- | --- | --- | --- |
| 1 | **AI Assisted** | Engineers use Claude ad hoc; nothing written down; context pasted by hand | Write `CLAUDE.md` and the standards file |
| 2 | **AI Contextualized** | `CLAUDE.md` + coding standards exist and are maintained; Claude follows house patterns | Adopt analyze → plan → self-review |
| 3 | **AI Workflow Driven** | Plans before code; self-review before human review; a fixed PR body; failure log in use | Connect Jira / GitHub / Figma |
| 4 | **AI Integrated** | Tickets read and moved through MCPs; PRs created by command; designs read from Figma | Add the AI table and story-point discipline |
| 5 | **AI Measured** | AI contribution on every PR; planned vs delivered per release; cycle time trusted because transitions come from commands | Review the numbers monthly; ask what they change |
| 6 | **AI Continuously Improved** | Commands and skills change from usage evidence; failures are logged and re-read; new patterns are shared back to this repo | — |

Xiangqi in September 2026 sits at level 5 with parts of 6 (the failure log and command
revisions) and gaps the shared repo now fills (plan review, failure-log friction, duplicated
commands).

---

## 6. Coaching playbook

**Session 0 — before meeting the team (1 h, alone).** Read their repo: is there a
`CLAUDE.md`? A standards file? A PR template? How do tickets move today? Who reviews? Note the
three most repeated chores in their recent PRs.

**Session 1 — context (90 min, whole team).** Show the journey in five minutes (section 2).
Together, write the NEVER list and the architecture map for `CLAUDE.md` — the team knows
these; the exercise is writing them down. Run `/setup-workflow` live; verify Jira ids live.
Homework: each engineer runs `/analyze-ticket` on their current ticket.

**Session 2 — workflow (60 min, one week later).** Review two plans the team produced. Ask:
did the plan change what you built? Run `/plan-review` on one of them in front of everyone —
the independent findings usually make the case. Agree the plan rule (`plans.requiredFor`).

**Session 3 — integration and PRs (60 min).** Turn on transitions and story points if the
team wants the board to follow the commands. Create one PR with `/create-pr`; agree the
weights; explain blank-not-zero and why Actual SP stays human.

**Session 4 — measurement and improvement (45 min, after a release).** Look at the release
view together. Ask what is still manual, which command nobody used, and what went wrong that
should be a failure-log entry. Send anything generic back to this repository as a PR.

**Things to say early**

- The AI % is an honest estimate, not a KPI, and never compares individuals.
- A plan is allowed to be short. A trivial fix is allowed to have none.
- Commands are for repeated multi-step chores; do not write one for a task done twice a year.
- If a command needs a project fact, it goes in `workflow.json`, not in the command.

---

## 7. Lessons learned (from Xiangqi, kept honest)

- **Worked:** written commands; plan + sign-off; quoting the ticket; the board following the
  commands; a failure log with a fixed format; discarding a misleading metric quickly.
- **Needed iteration:** the PR command grew in six steps and drifted between repos; plan
  sign-off lacked an independent view; the AI % was re-typed into Jira; failure logging
  competed with finishing the ticket.
- **MCP limits:** payload sizes, missing status history, connector grants, expiring OAuth,
  design labels vs swatches. Each has a fallback in the skills.
- **Still manual:** estimates, plan approval, merges, release process, product copy.
- **Measurement limits:** self-reported AI %; per-developer scoring misleads; cycle time is
  only as good as the transitions.

Full detail with evidence in [Xiangqi-AI-Journey.md](Xiangqi-AI-Journey.md).

---

## 8. Checklist before you call a team "adopted"

- [ ] `CLAUDE.md` and `coding-standards.md` exist, are reviewed like code, and are true.
- [ ] `.claude/workflow.json` exists; every id in it was verified live.
- [ ] Non-trivial tickets have a plan with a recorded approval.
- [ ] Self-review runs before human review; findings are visible in the PR or fixed.
- [ ] PR bodies follow the contract; the AI table is filled with blanks where honest.
- [ ] Planned SP comes from a human; Actual SP is never auto-filled.
- [ ] The failure log has been read by Claude in a session this month and written to.
- [ ] Nobody has forked a shared command; project facts live in config.
- [ ] The team can name one thing it changed because of what the numbers showed.
