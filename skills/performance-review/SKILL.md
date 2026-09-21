---
name: performance-review
description: Stack-agnostic performance checklist for analysis and review — request cost, data access, caching, hot paths, client rendering, payload/bundle size — with an impact-estimation format. Stack-specific rules come from the project's coding standards.
---

# Performance review

Read `project.stackSummary` and the performance section of `project.codingStandards` first; then
apply the checks below that fit the stack. Skip what does not apply — do not pad the report.

## Server / data

- Query count per request path; N+1 on relationships; missing eager loading.
- Unbounded reads (`SELECT *` without limit, `.all()` on growing tables, full scans); pagination.
- Indexes for new filter/sort columns; migrations on large tables run online.
- Cache opportunities and invalidation correctness; duplicated cache-key computation.
- Blocking calls in async/event handlers; synchronous I/O in hot loops.
- Batch operations (pipelines, bulk inserts) instead of per-item round trips.
- Connection pool pressure: new long-lived connections, pool size × worker count.
- Background jobs: idempotent, bounded batches, back-off on retry.

## Client / UI

- Re-render triggers: new object/array/function references created in render; missing
  memoisation where a child is expensive or frequently re-rendered.
- Selector/derived-data memoisation; stable references from state.
- Animations on compositor-only properties (`transform`, `opacity`), not layout properties.
- Event listeners and subscriptions cleaned up; no accumulating handlers.
- Dispatch/emit frequency: per keystroke, per scroll, per tick without throttling.
- Bundle: new dependencies, non-tree-shakeable imports, large libraries imported from the root.
- Low-end devices: blur/backdrop filters, large shadows on animated elements, heavy images.

## Impact estimate (per finding)

```
- <file:line> — <what>
  Frequency: <per request / per render / per second / on load>
  User impact: <latency / jank / memory / cost>
  Fix effort: <minimal change>
```

Prefer one measured number (query count, render count, payload size) over adjectives. If you
cannot measure, say "estimated" and how the estimate was made.
