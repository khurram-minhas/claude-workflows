---
name: review-standards
description: The shared code-review rubric and severity scale for self-review, independent review and PR review — scope vs plan, mechanical hygiene, correctness, error handling, patterns, tests, plus how to verify a reviewer's claims before acting. Stack-specific checks come from the project's coding standards file.
---

# Review standards

Two passes, always in this order. Read the actual files being changed and their neighbours
(the module's siblings, its tests, its callers) — never review a diff hunk in isolation.

## Pass 1 — Hygiene (mechanical, no judgement calls)

- **Scope vs plan:** every change maps to a plan step or is declared. Unrelated cleanups are
  flagged for removal or a separate PR.
- **Dead code:** grep for callers of every function touched; delete unused ones rather than
  leaving them "just in case". No commented-out code, no unreachable branches.
- **Debug leftovers:** console/print/debugger statements, temporary flags, hardcoded test data.
- **TODO/FIXME** added by this change: either done or turned into a ticket reference.
- **Duplicated work in a request path:** a cache key, a lookup, or an expensive derivation
  computed twice after a refactor. Count round trips before/after; tests will not catch this.
- **Misplaced logic:** code in a file whose stated responsibility it does not match.
- **Lint / format / typecheck** (`commands.*`) run on changed files; zero new warnings.
- **Suppressions** (`eslint-disable`, `noqa`, `type: ignore`) without a documented reason.

## Pass 2 — Engineering judgement

| Area | Ask |
| --- | --- |
| Correctness | Does the logic implement the acceptance criteria — and only them? Are state transitions complete (loading/success/failure/empty)? |
| Error handling | Are failures specific and handled where they can be handled? Nothing swallowed silently. |
| Concurrency | Retries, races, stale snapshots, idempotency — what happens if this runs twice? |
| Patterns | Does it use the codebase's existing utilities and idioms? Any new abstraction must have been asked for. |
| Contracts | API/schema/event changes are backward compatible or versioned; consumers updated. |
| Security | Run the `security-review` checklist on anything touching auth, input, secrets, money, PII. |
| Performance | Run the `performance-review` checklist on hot paths and user-visible latency. |
| Tests | Proportional to risk (`testing-standards`); a test that cannot fail is a finding. |
| Unenforced invariants | "Keep these in sync" comments and mirror-structures are findings: ask for a shared helper. |
| Readability | Names say what things are; early returns; no magic strings/numbers where a constant exists. |

Then apply the project's own checklist from `project.codingStandards` (stack rules, quality-gate
rules such as Sonar IDs, i18n, accessibility, styling conventions).

## Output format

Group findings by severity from `review.severityScale`:

```
🔴 Must fix before merge
- <file:line> — <what> → <why> → <suggested fix>
🟡 Should fix (tech debt if skipped)
🟢 Nice to have
```

Every finding cites a file and line. No finding without a suggested fix. Say explicitly when a
pass found nothing.

## Receiving review (from a human, an agent, or yourself)

Verify each claim against the codebase before acting — "no other callers", "matches the
existing pattern", "this is unused" are checkable. Fix what is real. Push back with technical
reasoning on what is not; never comply silently and never ignore silently. Apply this to a
subagent you dispatched exactly as to a human reviewer.
