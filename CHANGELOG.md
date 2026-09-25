# Changelog

## 0.2.0 — 2026-09-25

- New command `/parallel-tickets [KEY ...]`: one git worktree (detached at the base) and one
  interactive Claude session per ticket, 2–5 at a time; each session runs `/start-ticket` with
  every gate. Launchers: print (default), vscode, tmux.
- New skill `parallel-worktrees`.
- New config section `parallel` (`worktreeDir`, `launcher`, `maxTickets`).
- The "my ready queue" query moved from `/pick-ticket` into `jira-tracker`, shared by both
  commands.

## 0.1.0 — 2026-09-21

Initial extraction from the Xiangqi client and server setups.

- 14 commands: setup-workflow, pick-ticket, start-ticket, analyze-ticket, plan, plan-review,
  implement, self-review, perf-check, create-pr, pr-comments, ai-contribution, release-notes,
  log-failure.
- 13 skills: workflow-config, jira-tracker, github-pr, figma-design, planning-standards,
  review-standards, security-review, performance-review, testing-standards, git-standards,
  ai-contribution-scoring, ai-failure-log, human-in-the-loop.
- 2 agents: plan-reviewer, code-reviewer.
- Config schema + defaults; templates; examples for solo, React, Python and team layers.
- Docs: adoption guide, Xiangqi journey, command and skills catalogs, MCP and configuration
  guides, setup inventory.
