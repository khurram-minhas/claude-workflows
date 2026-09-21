---
description: Draft release notes for a version from the tracker's fixVersion and the merged pull requests between two git refs, grouped by type, with every line traceable to a ticket or PR. Drafts only — nothing is posted.
argument-hint: <version> [from-ref..to-ref]
---

Load `workflow-config`, `jira-tracker`, `github-pr`.

## Sources

- **Tracker** (if configured): `project = <projectKey> AND fixVersion = "<version>"` with
  `fields: ["summary", "issuetype", "status", "assignee"]`. Extract with `jq`.
- **Git**: `git log <from>..<to> --merges --oneline` and, for each merged PR,
  `gh pr view <n> --json title,body,labels,url` (title and the "Summary of changes" bullets
  only). If no range is given, use the previous tag to HEAD (`git describe --tags --abbrev=0`).

Cross-reference the two by ticket key. Report tickets with no merged PR and PRs with no ticket
as separate lists — they are the most useful output for a release manager.

## Output (markdown, ready to paste)

```
## <version> — <date>
### Features
- <TICKET> — <one line, user-facing wording> (PR #n)
### Fixes
### Improvements / internal
### Not shipped (in fixVersion, no merged PR)
### Unlinked PRs
```

Rules: one line per item, user-facing language for Features/Fixes, no commit hashes, nothing
that is not backed by a ticket or PR. Tickets whose status is not Done/Closed are marked
`(status: …)`. Do not post to the tracker, GitHub, or a docs system — hand the draft to the
user.
