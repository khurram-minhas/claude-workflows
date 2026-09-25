---
description: Work several tickets at once — create one git worktree per ticket (detached at the base) and open a separate interactive Claude session in each, which then runs /start-ticket with all its gates. Never writes to the tracker or writes code.
argument-hint: [TICKET ...]
---

Load `workflow-config`, `jira-tracker`, `parallel-worktrees`, `human-in-the-loop`.

This command prepares workspaces and opens sessions. Everything else — story points, status,
branch, plan, code, PR — happens inside each ticket's own session.

## Steps

1. **Resolve tickets.**
   - Keys in `$ARGUMENTS` → use them (deduplicated).
   - No keys and `tracker.type = jira` → run "My ready queue" (`jira-tracker`), show the
     table, and ask which tickets to run in parallel. Wait. Never pick for the user.
   - No keys and no Jira → say the queue needs Jira and ask for keys.
   - More than `parallel.maxTickets` → ask which to drop. Fewer than 2 → suggest
     `/start-ticket <KEY>` in this session instead, and stop unless the user insists.

2. **Resolve settings** — worktree root from `parallel.worktreeDir`, launcher from
   `parallel.launcher`, base from `git.baseBranch`. Check the launcher's binary now
   (`parallel-worktrees` → "Launchers"); if missing, plan to fall back to `print` and say so.

3. **Pre-check each ticket** (`parallel-worktrees` → "Creating one"): existing worktree path or
   an existing branch for the key → mark it *skipped* with the reason.

4. **Confirm** — show one table: ticket, worktree path, action (create / skip + reason), and
   the launcher. Ask to proceed. Wait for a yes; this is the gate.

5. **Create** — `git fetch origin <base>` once, then `git worktree add --detach` per ticket.
   A failure on one ticket is recorded and the loop continues.

6. **Launch** each created worktree with the launcher recipe from `parallel-worktrees`.

7. **Report** — one table: ticket, worktree path, result (launched / created-not-launched /
   skipped / failed + reason), and for `print` / `vscode` the exact command or prompt to use.
   Then:
   - the git-ignored files from the main checkout that the new worktrees lack;
   - "each session runs `/start-ticket`, which asks for story points and cuts the branch";
   - cleanup after merge: `git worktree remove "<path>"` — this command never removes one.

This command never writes to the tracker, never creates branches, and never writes code.
