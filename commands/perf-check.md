---
description: Performance-focused pass over the branch's changes — data access, request cost, caching, rendering, bundle size — with a frequency / user-impact / fix-effort estimate per finding.
argument-hint: [base-ref]
---

Load `workflow-config`, `performance-review`.

1. Diff = `git diff origin/<git.baseBranch>...HEAD` plus uncommitted changes (or the base in
   `$ARGUMENTS`). Read the changed files and their callers; identify which changes sit on a
   hot path (per request, per render, per event, per tick) versus one-off.
2. Read the performance section of `project.codingStandards` and `project.stackSummary`.
3. Apply the `performance-review` checklist — server/data items and client/UI items as they
   apply to this stack. Skip inapplicable sections silently.
4. Where you can measure, measure: count queries in a request path, count renders with a
   quick instrumented run, check bundle size delta if `commands.build` reports it.
5. Report each finding in the impact-estimate format, most impactful first, then a one-line
   verdict: safe to ship / fix 🔴 items first / needs a load test.

This command reports; it edits nothing unless the user asks for a specific fix.
