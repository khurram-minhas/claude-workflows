# The Xiangqi AI Journey — a case study

How one product team (a React web client and a Python/Flask API for an online board-game
platform, ~75k games/day) went from pasting Jira tickets into a chat window to a written,
measured, self-correcting engineering workflow — and what the rest of Arbisoft can take from it.

Everything in **Observed facts** is taken from the repositories' git history and files on
2026-09-21. **Team practices** is how the team says it works. **Lessons** and **Future
opportunities** are interpretation.

---

## 1. The arc

```
Manual Claude usage            (before Jun 2026)  ticket text pasted into chat; developer stitches the rest
        ↓
Project instructions           (2026-06-11)  CLAUDE.md + coding-standards.md + first 4 commands
        ↓
Reusable commands              (Jun 2026)    /analyze-ticket /review-pr /perf-check → /plan /create-pr /pick-ticket /ai-contribution
        ↓
Plans + failure log            (2026-06-19)  plan template, .claude/plans/, ai-failures.md
        ↓
Skills                         (server, Mar–Jun 2026)  domain skills (rules, engine, bot) + pre-review-checklist
        ↓
MCP integration                (Jun–Sep 2026) Jira reads → Jira writes (SP, transitions); Figma during analysis; Confluence sync
        ↓
Structured workflow            (2026-09-11)  /start-ticket makes the board follow the commands
        ↓
Human review gates             (throughout)  ticket pick, SP, plan sign-off, PR review, thread resolution
        ↓
Dev metrics                    (Jul–Aug 2026) AI Contribution Checklist on every PR; Jira-centric delivery dashboard
        ↓
Continuous improvement         (ongoing)     ai-failures.md, 6 revisions of /create-pr, plan "AI failure points"
```

---

## 2. Observed facts

### Where it started

Before June 2026 there was no Claude configuration in either repository. Usage was the
default: a Jira requirement copied into Claude, help requested, and the developer performing
analysis, planning, PR text, board updates and documentation by hand. Nothing was written
down, so nothing was repeatable and nothing could be audited later.

### Timeline (from git)

| Date | Repo | Change |
| --- | --- | --- |
| 2026-03-25 | server | First Claude helper files added alongside a Redis library upgrade (PR #1795) |
| 2026-06-11 | client | `CLAUDE.md`, `coding-standards.md`, `/analyze-ticket`, `/review-pr`, `/perf-check` (PR #3826) |
| 2026-06-19 | client | Plan template, `.claude/plans/`, `ai-failures.md`, `/plan`, `/create-pr` (PRs #3853, #3858) |
| 2026-06-23 | both | AI Contribution Checklist in the PR template; `/ai-contribution`; `/pick-ticket` (PRs #3867, #3870, #1882) |
| 2026-07-24 | client | Confluence auto-sync of plans/agents/commands from `/create-pr` (PR #3961); `/add-theme` + agents |
| 2026-08-03 | server | Confluence sync for commands/skills/docs (PR #1909) |
| 2026-09-02 | client | `/pr-review-comments`, `/request-translations`, `/apply-translations` (PR #3903) |
| 2026-09-11 | client | `/start-ticket`: Story Points + In Progress transition moved into a command (PR #4094) |

### Volume

- 62 plan files on the client between 2026-06-19 and 2026-09-11 (6 in June, 19 in July, 30 in
  August, 7 in the first third of September); 8 plans + 4 specs + 2 runbooks on the server.
- 13 of the client plans carry filled-in "AI Failure Points" (46 entries); 5 entries promoted
  to the repo-wide log.
- `/create-pr` has 6 revisions — the most of any command. `/analyze-ticket`, `/review-pr`,
  `/perf-check` have not changed since day one.
- 12 commands on the client, 7 on the server; 6 of them exist in both with diverging copies.
- 7 server skills (6 domain knowledge, 1 review hygiene); 0 client skills.
- 2 client agents (theme and board creation), both product-specific.

### Integrations

- **Jira** via the Atlassian MCP (user scope): read for pick/analyse, write for Story Points and
  two transitions. Field ids (`customfield_10010`, `_10833`, `_11729`) and transition ids (31,
  51) were verified live and hard-coded into the prompts.
- **GitHub** entirely via `gh`; no GitHub MCP.
- **Figma** MCP registered on the client only; used ad hoc — 7 plans cite Figma node ids, and
  `/add-theme` reads swatches from it. No command requires it.
- **Confluence**: 92 pages (73 client, 19 server) mapped from plan/command/agent/skill/doc files through a committed
  `confluence-map.json`.
- **SonarQube** runs in CI on every PR; the commands reference its rule ids by name.

### Measurement

Every PR body carries an AI Contribution Checklist (weighted activities, blank-not-zero, single
percentage). Planned SP is set on the ticket by `/start-ticket`; Actual SP and the AI % on the
ticket are filled in by hand. A separate Node project (`dev-metrics-dashboard`) reads Jira
(SP, AP, AI %) and GitHub PRs and renders release progress, In Progress → Code Review cycle
time and per-developer delivery — with **no composite score and no ranking**.

---

## 3. Team practices (how the workflow runs today)

```
/pick-ticket   → table of my "Selected for Development" tickets on the active release; I choose
/start-ticket  → ticket shown in full; SP asked (1/2/3/5) and written; → In Progress; branch cut
/plan          → runs /analyze-ticket, scaffolds .claude/plans/XQ-NNNN-slug.md, asks for sign-off
implement      → conversational, plan open, standards loaded via CLAUDE.md
/review-pr, /perf-check → self-review before anyone else sees the diff
/create-pr     → fixed PR body, AI table, → Code Review, labels/reviewers, Confluence sync
human review   → GitHub; /pr-review-comments to work the threads
merge          → human; Jira AI % typed in by hand; dashboard picks it up
```

Rules the team holds to (written in both `CLAUDE.md` files): never move a ticket backwards;
never invent story points; never write Actual SP from a command; report failed tracker calls
next to successes; plan before code for anything non-trivial; log corrections while fresh.

### Examples from the repository

- **A plan that found the root cause before code** — `XQ-5167` (mobile back button → Not Found):
  the plan proves the cause with `resolvePath('./lobby', '/tournaments')`, names the two back
  buttons, picks the fix that matches the house pattern, and lists an invalid-HTML issue as
  out of scope for its own ticket. Two-line code change; the reasoning outlives the branch.
- **Self-review catching real defects** — `XQ-5043` (theme-button unread indicator): four
  issues found before the PR opened, including a guest-state leak and a repeat of a colour
  bug fixed one ticket earlier.
- **A failure entry that changed a rule** — `XQ-5114`: a `git stash` mid-task collided with an
  unrelated stash and produced a false pass. The correction ("copy to scratchpad, never
  stash") is now in `ai-failures.md` and in this repo's `git-standards` skill.
- **A metric discarded for being wrong** — the dashboard's first version scored developers on
  PR speed and review turnaround; on real data a developer with 12 PRs outscored people who
  delivered more. It was rebuilt around Jira SP/AP with no score at all.
- **Domain knowledge as skills** — the server's `xiangqi-knowledge` router points at six
  focused skills (board geometry, engine protocol, adjudication rules, bot logic, trade
  detection, testing) each with trigger keywords in its description, so Claude loads the right
  one without being told.

---

## 4. Lessons learned

**What worked**

- *Writing the workflow down as files.* A command is a habit that survives a change of person
  or a context reset. Six revisions of `/create-pr` were possible because it was a file.
- *Plan first, sign off, then code.* The plan's "Out of scope" section turned scope creep into
  something that has to be argued for.
- *Grounding in the real ticket and the real code.* Every analysis command says "read the
  files first; do not write code". Quoting the ticket description instead of paraphrasing it
  made estimates and plans arguable.
- *Blank-not-zero and single-percentage rules for the AI score.* Small rules, but they are
  what make tickets comparable across a release.
- *Letting the commands own the board.* `/start-ticket` and `/create-pr` moved the ticket;
  cycle-time data became trustworthy because the transitions happened when the work did.
- *A failure log with a fixed format.* Entries are specific enough to be re-read before the
  next similar task.

**What required iteration**

- The PR command grew by accretion (Jira transition, then labels/reviewers, then Confluence
  sync) and its client/server copies drifted (title style, weight table). → the shared repo
  moves every fact to config and keeps one copy.
- Sign-off was the author reading their own plan. → `/plan-review` with an independent
  agent context.
- The AI % is typed into Jira by hand from the PR. → optional write-back from `/create-pr`.
- Failure logging happened on roughly one plan in five. → `/log-failure` makes it a
  two-second action; `/implement` and `/self-review` prompt for it.

**Commands that earned their place:** plan, create-pr, start-ticket, review-pr, pick-ticket.
**Commands that were nice but rarely needed:** perf-check as a separate step (usually folded
into review). **Commands that were never built although discussed:** plan review, release
notes.

**MCP limitations met:** Jira payloads too large for context (solved with `jq` on the saved
file); the bulk search lacking status history (solved outside Claude); Confluence exposing
only non-engineering spaces at first; OAuth sessions expiring in non-interactive runs; Figma
labels disagreeing with swatches.

**Context problems met:** hallucinated tokens and keys (hence "verify the key exists"), stale
mental models after a refactor (hence "read the file first, every time"), and false passes from
tooling misuse (the stash incident).

**Cases that still need a human:** every estimate, every plan approval, every merge, product
copy, anything touching money or identity, and the judgement of whether a plan is one PR or
three.

**Measurement limitations:** the AI % is self-reported and retrospective; per-developer
comparisons were found to mislead and were removed; cycle time is only as honest as the
transitions, which is why they were moved into commands.

---

## 5. Future opportunities

1. Adopt the shared repository in both Xiangqi repos and delete the six duplicated commands,
   keeping only the product-specific ones (`/request-translations`, `/apply-translations`,
   `/add-theme`, the two agents).
2. Turn `/plan-review` on for anything above 3 SP; keep a note of how often it changes a plan.
3. Enable `aiContribution.writeToTracker` once the team agrees the PR table is the source of
   truth, and retire the manual re-typing.
4. Write the release checklist down; if it stabilises, encode it as a command.
5. Move the stack-specific sections of `coding-standards.md` to the top of the file and the
   "core rules" to the shared template, so new joiners read the codebase-specific part first.

No productivity claim is made here. The repository shows *what changed in how work is done*;
whether it made the team faster is a question for the delivery dashboard over several
releases, not for this document.
