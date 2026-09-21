---
name: github-pr
description: GitHub rules for shared commands using the gh CLI — the PR body contract, idempotent assignee/label/reviewer edits, reading and resolving review threads via GraphQL, and deriving owner/repo safely. No GitHub MCP required.
---

# GitHub pull requests via `gh`

All GitHub work goes through the `gh` CLI. Check once per command that it is authenticated:

```bash
gh auth status -h github.com 2>&1 | head -3
```

If it is not, stop the GitHub steps and say so; do not fall back to raw `git push` plus a
hand-written URL.

## Deriving the repository

Prefer, in order: `github.repo` from config → the `--repo` in a PR URL the user gave →
`gh repo view --json nameWithOwner -q .nameWithOwner`. Remotes can redirect (renamed orgs), so a
URL given by the user beats the `origin` remote.

## PR body contract

The body is fixed so PRs are comparable across a team and machine-readable by dashboards:

```
[<TICKET>](<tracker.browseUrl><TICKET>)

#### Summary of changes
- <2–5 bullets, 1–3 lines each, each a real change grounded in the diff>

#### AI Contribution Checklist
<table from the ai-contribution-scoring skill>

**AI Contribution Score:** <N>%

**Planned SP:** <read from tracker, or _fill in from tracker_>
**Actual SP:** _fill in manually_

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

Rules: first line is the ticket link alone; no commit hashes; no "misc updates"; no sub-bullets;
if there is no tracker, the first line is omitted and the SP lines are dropped. If the repo has
its own `.github/PULL_REQUEST_TEMPLATE.md`, follow its section order and add the AI table where
the template puts it.

Title: `git.prTitlePattern`, e.g. `ABC-123: Add rate limit to invite endpoint`.

## Assignee, labels, reviewers (idempotent)

Read state first, then add only what is missing, then report what changed:

```bash
gh api user -q .login
gh pr view <PR#> --repo <owner/repo> --json author,assignees,labels,reviewRequests
gh pr edit <PR#> --repo <owner/repo> --add-assignee @me
gh pr edit <PR#> --repo <owner/repo> --add-label "<github.labels.readyForReview>"
gh pr edit <PR#> --repo <owner/repo> --add-reviewer a,b
```

- `--add-*` never removes anything; re-adding is a no-op. Still read first so the report is
  honest about what was already there.
- Reviewer policy `pool-except-author`: request everyone in `github.reviewerPool` except the PR
  author and anyone already requested. **Never include the author** — GitHub rejects the whole
  call, taking valid reviewers with it.
- A missing label or non-collaborator login is reported plainly, not silently skipped.

## Review threads

List unresolved threads (GraphQL — REST does not expose resolution state):

```bash
gh api graphql -f query='query($o:String!,$r:String!,$n:Int!){repository(owner:$o,name:$r){pullRequest(number:$n){reviewThreads(first:100){nodes{id isResolved path line comments(first:20){nodes{author{login} body url}}}}}}}' -F o=<owner> -F r=<repo> -F n=<PR#>
```

Resolve one (only after the user approves the implemented change):

```bash
gh api graphql -f query='mutation($id:ID!){resolveReviewThread(input:{threadId:$id}){thread{isResolved}}}' -F id=<threadId>
```

Reply in a thread: `gh api repos/<owner>/<repo>/pulls/<PR#>/comments/<commentId>/replies -f body='...'`.

## Pushing

Push with `-u` if the branch has no upstream or has unpushed commits. Never force-push from a
command. Never push from `git.baseBranch`.
