# Skills Catalog

Skills hold the rules; commands hold the sequence. A rule lives in exactly one skill and is
referenced by every command that needs it.

```
                 workflow-config ──────────── every command
                 human-in-the-loop ────────── every command that acts outwardly
 jira-tracker ── pick, start, analyze, create-pr, release-notes
 github-pr ───── create-pr, pr-comments, release-notes
 figma-design ── analyze (optional)
 git-standards ─ start, implement, create-pr, pr-comments
 planning-standards ── plan, plan-review, implement, plan-reviewer agent
 review-standards ──── plan-review, self-review, pr-comments, code-reviewer agent
 security-review ───── analyze, self-review, agents
 performance-review ── analyze, self-review, perf-check, agents
 testing-standards ─── plan, implement, self-review
 ai-contribution-scoring ── create-pr, ai-contribution
 ai-failure-log ─────────── analyze, implement, self-review, log-failure
```

| Skill | What it owns | Where it came from |
| --- | --- | --- |
| `workflow-config` | Resolution order of config layers, null-means-skip, never-guess-ids, read the project standards | New — the mechanism that makes commands portable |
| `jira-tracker` | Tool map, jq extraction, ticket presentation, never-backwards, never-invent-SP, paste fallback | Xiangqi `start-ticket` / `create-pr` / `apply-translations` rules, generalised |
| `github-pr` | `gh` usage, PR body contract, idempotent metadata edits, GraphQL threads, repo derivation | Xiangqi `create-pr` / `pr-review-comments` |
| `figma-design` | Which Figma tools for which need, token mapping, recording links in the plan | Xiangqi plans and `add-theme` (ad hoc use, now written down) |
| `planning-standards` | When a plan is required, sections, sign-off gate, plan review rubric | Xiangqi `_template.md`, memory note "plans-first", the plan-review gap |
| `review-standards` | Two-pass review, severity scale, output format, verifying reviewer claims | Xiangqi `review-pr` (both repos) + server `pre-review-checklist` |
| `security-review` | Trigger conditions, checklist, escalate-don't-decide | Server `review-pr` item 5, expanded |
| `performance-review` | Server/data and client/UI checklists, impact-estimate format | Xiangqi `perf-check` (both repos) |
| `testing-standards` | Policy modes, regression rule, safe test runs, what a test asserts | Xiangqi template's regression rule, server test-safety rules, `/write-tests` |
| `git-standards` | Branching, commits, dirty tree, never stash mid-task | Xiangqi `start-ticket` + the XQ-5114 failure entry |
| `ai-contribution-scoring` | Weights from config, blank-not-zero, formula, honesty statements | Xiangqi `create-pr` / `ai-contribution` |
| `ai-failure-log` | Two tiers, entry format, when to write, when to read | Xiangqi `ai-failures.md` + plan section |
| `human-in-the-loop` | Responsibilities split, gates table, escalation, honesty rules | Xiangqi `CLAUDE.md` rules scattered across commands, consolidated |

## What is deliberately *not* a shared skill

- **Stack standards** (React, Flask, …). They belong to the project's
  `.claude/coding-standards.md`; the shared repo ships starters under
  `templates/coding-standards/`. Every reviewing/planning command reads that file.
- **Domain knowledge** (game rules, business rules). The reference project's server keeps a
  router skill (`xiangqi-knowledge`) pointing at focused domain skills with trigger keywords.
  That *pattern* is recommended: one router, several focused `skills/<topic>/SKILL.md`, each
  with a `description` that lists the trigger terms so Claude loads it unprompted.

## Writing a new shared skill

1. It must be needed by at least two commands or be a rule teams keep re-explaining.
2. One responsibility; under ~120 lines; no project names, keys, or people.
3. `description` says *when* to load it, not just what it is.
4. Update this catalog and the command(s) that now reference it.
