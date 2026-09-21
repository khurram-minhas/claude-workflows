---
name: git-standards
description: Branch, commit and working-tree rules for shared commands — branch naming from config, cutting from an up-to-date base, dirty-tree handling, never stashing mid-task, commit message format, what never to do from a command.
---

# Git standards

## Branches

- Cut from an up-to-date base: `git fetch origin <git.baseBranch>` then
  `git checkout -b <branch> origin/<git.baseBranch>`.
- Name from `git.branchPattern` (default `<name>/<TICKET>-<slug>`). Look at recent branches
  (`git branch -r | head`) and match the repo's real convention if it differs from config —
  then suggest fixing the config.
- If the working tree is dirty or a branch for the ticket already exists: **stop and tell the
  user**. Do not stash, do not force-create, do not switch with uncommitted changes.

## Commits

- Title from `git.commitTitlePattern` (default `<TICKET>: <summary>`), imperative, ≤ 72 chars.
- Body explains *why* when it is not obvious from the title.
- Commit only when the user asks. Never commit on `git.baseBranch`.
- Never `--amend` or rebase someone else's commits; never force-push from a command.

## Working tree discipline

- **Never `git stash` mid-task** to test "does this change matter". Repos accumulate unrelated
  stashes; a pop can collide, pollute the tree with someone else's WIP and produce a false pass.
  Copy the file to the scratchpad, edit in place, restore from the copy.
- Never `git checkout -- <file>` / `git restore` on files you did not change in this task.
- Diff against the merge-base, not the base tip: `git diff origin/<base>...HEAD`.
- Before claiming a change "has no effect elsewhere", grep for callers.

## Ticket key from branch

`<projectKey>-\d+` anywhere in the branch name (`jane/ABC-123-thing` → `ABC-123`). If the
branch has none and none was given, ask.
