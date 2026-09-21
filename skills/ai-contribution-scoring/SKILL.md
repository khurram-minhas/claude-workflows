---
name: ai-contribution-scoring
description: How to estimate and present the AI Contribution Checklist for a ticket — weights from config, blank-not-zero rule, the score formula, the honesty statements that must accompany it, and what is never auto-filled (Planned SP, Actual SP). Used by /create-pr and /ai-contribution.
---

# AI contribution scoring

A retrospective, self-reported estimate of how much of each activity was AI-assisted. It is
**not a measurement** and must never be presented as one.

## Weights

From `aiContribution.weights` (default nine rows summing to 100). A project removes rows it never
does — e.g. a repo that does not write unit tests drops "Unit Tests" and redistributes weight.
The table always prints exactly the configured rows, in configured order.

## Scoring procedure

1. Look back over **this conversation** (and `git log`/`git diff` against the base if there are
   commits) for everything done toward the ticket.
2. For each activity, judge the percentage that was AI-assisted (0–100), grounded in what
   actually happened — not a default split, not a flattering one.
3. **Blank, never zero:** an activity that did not happen in this thread is left blank and its
   weight leaves the denominator. Zero means "happened, with no AI help".
4. Row score = weight × AI% ÷ 100.
5. **Score = Σ filled row scores ÷ Σ weights of filled rows × 100**, rounded to a whole
   percent. Always a single 0–100 figure — never a fraction like `81.75/90`.

## Table

```
| Activity | Weight | AI Assisted (%) | Score (Weight × AI%) |
| --- | --- | --- | --- |
| <row per configured weight> | <w> | <% or blank> | <score or blank> |
| **Total** | **<Σ weights>** | | |

**AI Contribution Score:** <N>%
```

## Statements that always accompany the table

- "Retrospective estimate based on this conversation, not a precise measurement."
- Planned SP is **read** from the tracker (set by `/start-ticket` from a human's answer), never
  estimated here. Actual SP is always a manual fill-in.
- If `aiContribution.writeToTracker` is false (default), the number is not written anywhere
  except the PR body; say where a human should record it if the team tracks it.

## Why the rules are strict

The number feeds delivery dashboards that compare tickets and releases. Zero-instead-of-blank
punishes tickets that needed no refactoring; fractions instead of percentages make releases
incomparable; estimating SP here would let the AI grade its own homework.
