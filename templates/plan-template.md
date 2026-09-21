# <TICKET> — <Short title>

**Ticket:** [<TICKET>](<browse-url><TICKET>) · <type> · <priority> · fixVersion <x.y.z> · Planned SP <from tracker>
**Branch:** `<branch>`
**Date:** YYYY-MM-DD
**Design:** <Figma / spec links, or "none">
**Approved:** _<name, date — filled in by the human who signs off>_

---

## Ticket

> Quote the requirement as written: repro steps, expected vs actual, acceptance criteria.

## Goal

One paragraph. What problem this solves and for whom.

## Root cause (bugs) / Approach (features)

Grounded in `file:line` references that were actually read. For bugs: how the cause was
proven, not just the first plausible story.

## Files to create or modify

| File | Action | Notes |
| --- | --- | --- |
| `path/to/file` | modify | what changes |

## Architecture impact

Only rows that apply. Delete the rest.

| Layer | Impact |
| --- | --- |
| Data model / migrations | |
| API / event contracts | |
| State management | |
| Async / background work | |
| Real-time events | |
| i18n | |
| Styling / design tokens | |
| Config / feature flags | |
| Analytics | |

## Edge cases and risks

- Concurrency / retries / partial failure:
- Security-sensitive surface touched? (auth, input, secrets, money, PII) — yes/no, and what:
- Quality gate / lint rules likely to bite:

## Test strategy

- Automated: <what, which file>
- Manual: <exact steps, viewport / account state / data>
- Regression test (bug fixes — required): failure scenario reproduced: … · test file: …
  (feature work: `N/A — feature work`)

## Rollout / rollback

Plain deploy; revert = revert the PR. — or — flag / migration / contract details.

## Out of scope

Explicit, so scope creep has to be argued for.

- 

## Steps

- [ ] 1.
- [ ] 2.

## Done when

- [ ] 
- [ ] 

## AI failure points

Filled in during and after the work. Format: `- [<TICKET>] what the AI did wrong → what the correct approach was`

- 
