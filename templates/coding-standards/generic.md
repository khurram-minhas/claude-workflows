# <Project> — Coding Standards

The shared commands (`/analyze-ticket`, `/plan`, `/implement`, `/self-review`, `/perf-check`)
read this file. Part 1 is stack-agnostic and should stay. Part 2 is yours to fill with the
rules that are true for this codebase — keep it to rules that have actually caused bugs or
review churn.

---

## Part 1 — Core AI engineering rules

### 1. Never assume architecture
Inspect the surrounding code (the module's siblings, its callers, its tests) and understand who
owns which state before proposing anything. Do not introduce isolated patterns that conflict
with what exists.

### 2. Prefer existing project patterns
Existing utilities, helpers, hooks, API clients and conventions win. No new abstractions
unless explicitly requested.

### 3. Hallucination prevention
Never invent endpoints, functions, selectors, config keys, translation keys, props, or backend
contracts. If unsure: state the assumption, ask, or give a guarded suggestion.

### 4. Performance-safe by default
Know which paths are hot (per request, per render, per event). Do not add work to them without
saying so. See the performance section below for the stack specifics.

### 5. Concurrency and failure modes are part of correctness
Explain what happens on retry, on a race, on partial failure. Idempotency is a requirement for
anything that can run twice.

### 6. Refactor rules
Preserve behaviour first; explain the reasoning; name the regression risk; no unrelated
cleanup; minimal diff; large refactors are incremental.

### 7. Verification requirement
AI-generated output is verified before merge: run the lint/test commands, name edge cases,
mention uncertainty, highlight assumptions. "Done" means the proof was seen.

### 8. Human decisions stay human
Requirements, architecture, estimates, security-sensitive choices and product trade-offs are
presented as options with a recommendation, never decided silently.

---

## Part 2 — This codebase

### Structure
<Directory layout, the module pattern, where each kind of code lives.>

### Naming and style
<Conventions the linter does not enforce.>

### Data access / state
<Ownership rules, invariants, things that must go through a specific layer.>

### Error handling
<Exception hierarchy, what is caught where, what is never swallowed.>

### Performance
<Hot paths, known limits, patterns that are banned on them.>

### i18n / accessibility (if applicable)
<>

### Lint and quality gate
<Commands to run; rules that commonly trip new code, with IDs.>

### Testing
<Framework, fixtures, how to run one file safely, what must never be run.>

### Lessons from code review
<Recurring review comments, each as a rule with a one-line why.>
