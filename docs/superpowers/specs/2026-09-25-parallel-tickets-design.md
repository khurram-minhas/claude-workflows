# /parallel-tickets — design

## Goal

Work 2–5 tickets at the same time, each in its own git worktree and its own interactive Claude
session, while the engineer's current chat stays free.

## Decisions

- **One interactive session per worktree.** Each session runs the normal lifecycle
  (`/start-ticket` → `/plan` → `/implement` → `/create-pr`) with every existing gate. No
  background or headless runs.
- **The command only prepares and launches.** It never writes to the tracker and never writes
  code. Its one gate is confirming the ticket list and worktree paths.
- **Worktrees start detached at `origin/<git.baseBranch>`.** `/start-ticket` inside the worktree
  then cuts the branch exactly as it does today, so branch naming stays in one place and
  `start-ticket` needs no change.
- **Launcher is configurable:** `print` (default, paste-ready commands), `vscode`
  (`code -n <path>`), `tmux` (one window per ticket running `claude "/start-ticket KEY"`).
  A missing binary falls back to `print` for that run.
- **Out of scope for v1:** copying git-ignored files / running install in new worktrees, a
  status/cleanup command, per-worktree port isolation. The report reminds the engineer that
  git-ignored files are absent. Worktree removal is manual (`git worktree remove`).

## Pieces

| Piece | Change |
| --- | --- |
| `commands/parallel-tickets.md` | New command: resolve tickets (args or multi-pick) → confirm → create worktrees → launch → report |
| `skills/parallel-worktrees/SKILL.md` | New skill: location, detached start, launcher recipes, one-session-per-worktree, never remove |
| `skills/jira-tracker` | Gains the "my ready queue" query, now shared by `/pick-ticket` and `/parallel-tickets` |
| `skills/human-in-the-loop` | Gates table gains the parallel-sessions row |
| Config | `parallel.worktreeDir` (`../<repo>-worktrees`), `parallel.launcher` (`print`), `parallel.maxTickets` (5) |
| Docs | Command/Skills catalogs, Configuration Guide, README; version bump to 0.2.0 |

## Error handling

- Tracker not Jira → keys must be passed; say so.
- Path or branch for a ticket already exists → skip that ticket, report it; never force.
- More keys than `maxTickets` → ask which to drop.
- Dirty main checkout is irrelevant: worktrees come from `origin/<base>`.

## Testing

Add the plugin as a local marketplace in a real repository, run `/parallel-tickets` with 2–3
real tickets under each launcher, confirm each session's `/start-ticket` cuts its branch in
its own worktree.
