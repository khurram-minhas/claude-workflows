---
name: ai-failure-log
description: The two-tier AI failure log — per-plan "AI Failure Points" and the repo-wide failures file — with the fixed entry format, when to write an entry, and how entries get promoted so the same mistake is not made twice. Used by /implement, /self-review, /log-failure and read at the start of analysis/planning.
---

# AI failure log

Purpose: stop the same mistake twice. It is curated, not scored — there is no failure metric.

## Two tiers

1. **Per ticket** — the plan's `## AI failure points` section, written while the correction is
   fresh.
2. **Repo-wide** — `failureLog.path` (default `.claude/ai-failures.md`), bucketed by area
   (`failureLog.areas`), holding entries that generalise beyond one ticket.

## Entry format (fixed)

```
- [<TICKET>] <what the AI did wrong> → <what the correct approach was>
```

Specific beats general: name the file, the function, the wrong assumption and the check that
would have caught it.

## When to write one

- The user corrected a wrong assumption, a wrong file, a wrong pattern.
- A verification (test, lint, manual check) failed for a reason the AI should have foreseen.
- The AI reported success that turned out false.

Write it immediately in the plan (tier 1). At `/self-review` or `/create-pr` time, ask once:
"Any of these worth promoting to the repo-wide log?" and move the ones that apply beyond this
ticket (tier 2), keeping the ticket reference.

## Reading it

`/analyze-ticket`, `/plan` and `/implement` read the repo-wide log at the start and quote any
entry whose area overlaps the current work. That is the whole mechanism — no automation, just
reading before acting.
