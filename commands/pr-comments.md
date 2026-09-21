---
description: Work through the unresolved review threads on a pull request — implement each requested change locally, show the diff, and resolve the thread only after the user approves. Answers questions instead of changing code when a reviewer asked one.
argument-hint: <PR URL | PR number>
---

Load `workflow-config`, `github-pr`, `review-standards`, `git-standards`.

## Phase 1 — Fetch

Parse `$ARGUMENTS` into owner/repo and PR number (a URL beats the origin remote). Fetch all
unresolved review threads (`github-pr` → GraphQL). Sort by path then line. Show a table:
`# | reviewer | file:line | comment (first line)`, the PR title, and the count. Ask "Start
reviewing these?" and wait.

## Phase 2 — One thread at a time

For each thread:

1. Show reviewer, file:line, full comment.
2. Read the code and its surroundings; understand the intent before editing.
3. If the comment is a **question**, draft an answer instead of a change.
4. If the request would introduce a bug or contradict the plan/standards, say so with
   reasoning and ask how to proceed — never comply blindly, never ignore.
5. Implement the minimal change. Run `commands.lint` (and the relevant single test file).
   Show the diff.
6. Ask the user to choose: **apply & resolve** · **apply only** · **revise** · **reply only**
   (post text they provide) · **skip** · **stop**.
7. Resolve the thread only for "apply & resolve" or when the user explicitly asks to resolve
   without a change. Reply in-thread with one line saying what was done when resolving.

## Phase 3 — Summary

Threads total / changed / resolved / replied / skipped / remaining. Then offer: review the
combined diff, commit (`git-standards` title pattern), push, or stop. Do none of these without
a yes.
