---
name: parallel-worktrees
description: Rules for running several tickets at once — one git worktree and one interactive Claude session per ticket, where worktrees live, why they start detached at the base, the launcher recipes (print / vscode / tmux), and what a command must never do to a worktree. Used by /parallel-tickets.
---

# Parallel worktrees

## Model

One ticket → one worktree → one interactive Claude session. Each session runs the normal
lifecycle (`/start-ticket` → `/plan` → `/implement` → `/create-pr`) with every gate intact.
Nothing runs in the background on the engineer's behalf; the command that sets this up only
prepares worktrees and opens sessions.

- Never run two sessions in the same worktree — they will edit the same files.
- Never switch a worktree to another ticket's branch; create another worktree instead.

## Where worktrees go

- Root: `parallel.worktreeDir`, default `../<repo>-worktrees`, where `<repo>` is the basename
  of `git rev-parse --show-toplevel`. Relative paths resolve from the repository root.
  Outside the repository, so tools that walk the tree (lint, test, search) never see them.
- One directory per ticket: `<root>/<TICKET>`.

## Creating one

```bash
git fetch origin <git.baseBranch>            # once, before the loop
git worktree add --detach "<root>/<TICKET>" "origin/<git.baseBranch>"
```

Start **detached** at the base. `/start-ticket` in that worktree then cuts the branch per
`git-standards` (same name pattern, same base) — branch rules stay in one place.

Skip the ticket and report it, never force, when:
- `<root>/<TICKET>` already exists (`git worktree list` shows it, or the directory is there);
- a local or remote branch already contains the ticket key
  (`git branch -a --list "*<TICKET>*"`) — the ticket was started elsewhere.

## What a new worktree does not have

Git-ignored files are not copied: `.claude/workflow.local.json`, `.env*`, dependency folders
(`node_modules`, virtualenvs), build output. Say so in the report, naming the ones that exist
in the main checkout (check `.claude/workflow.local.json`, `.env*`, `node_modules`, `.venv`,
`venv` at the repository root). Do not copy or install anything unless the user asks.

## Launchers (`parallel.launcher`)

The session prompt is `/start-ticket <TICKET>`. If the plugin's commands appear namespaced in
this session, use `/arbisoft-workflows:start-ticket <TICKET>` instead.

| Launcher | Action per ticket | Needs |
| --- | --- | --- |
| `print` (default) | Print `cd "<path>" && claude "/start-ticket <TICKET>"` | nothing |
| `vscode` | `code -n "<path>"`, then print "in the new window, open Claude and run `/start-ticket <TICKET>`" — the extension cannot be started with a prompt | `code` on PATH |
| `tmux` | Inside tmux (`$TMUX` set): `tmux new-window -n <TICKET> -c "<path>" 'claude "/start-ticket <TICKET>"'`. Outside: first ticket `tmux new-session -d -s <repo>-parallel -n <TICKET> -c "<path>" '…'`, rest `new-window -t <repo>-parallel …`; print `tmux attach -t <repo>-parallel` | `tmux` on PATH |

If the launcher's binary is missing (`command -v code` / `command -v tmux` fails), fall back to
`print` for this run and say why. An unknown launcher value is treated the same way.

## Never, from a command

- `git worktree remove`, `git worktree prune`, `rm -rf` on a worktree, or deleting its branch.
  Cleanup is the engineer's: `git worktree remove "<path>"` after the PR merges.
- Create more than `parallel.maxTickets` worktrees in one run.
- Write to the tracker. Status and story points belong to each session's `/start-ticket`.
