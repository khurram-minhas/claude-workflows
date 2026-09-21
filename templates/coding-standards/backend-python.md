# <Project> — Coding Standards (Python service starter)

Derived from a production Flask / SQLAlchemy / Redis / Celery service. Part 1 is
stack-agnostic. Part 2 is a **starting point** — delete what is not true here and add what is.

---

## Part 1 — Core AI engineering rules

1. **Never assume architecture** — read the model, the helper layer, the handler and the task
   that touch the same data before proposing anything.
2. **Prefer existing project patterns** — existing helpers, schemas, query methods, task
   patterns; no new abstractions unless asked.
3. **Hallucination prevention** — never invent endpoints, model fields, config keys, cache keys.
4. **Performance-safe by default** — know the hot paths and the request budget; count queries.
5. **Concurrency and failure modes** — retries, races, stale snapshots, idempotency are part of
   correctness, not polish.
6. **Refactor rules** — behaviour first, minimal diff, incremental.
7. **Verification requirement** — lint and run before "done"; name edge cases and assumptions.
8. **Human decisions stay human** — schema changes, data retention, permissions: options with a
   recommendation.

---

## Part 2 — This codebase (edit me)

### Structure
- `api/` routes and namespaces · `sockets/` event handlers · `models/` ORM · `helpers/`
  utilities · `tasks/` background jobs · `exceptions.py` hierarchy.

### Style
- Type hints on every function (modern syntax: `list[str]`).
- Specific exceptions only — never bare `except:`.
- Validation lives in the schema layer (validators), handlers are pass-throughs after `load()`.
- Response shapes map to a schema/model, never a hand-built dict.
- Complex DB operations (upserts, conflict handling) are model/query-layer methods.

### Data access
- Eager-load relationships; zero tolerance for N+1.
- No unbounded `.all()` on growing tables — paginate, batch.
- Redis-only logic lives in `helpers/`, not in database helper modules.
- Never blind-save a whole cached document that another process may have updated; patch
  fields or use compare-and-set.

### Background work
- Tasks are idempotent and bounded; batch and self-chain rather than loop unbounded.
- Real-time event handlers have ack timeouts; no synchronous blocking in async handlers.

### Migrations
- Schema change + rollback plan always. Large backfills run outside the migration transaction
  in batched, keyset-paginated commands.

### Lint and quality gate
- `<lint command>` clean on changed files. `<quality gate rules that bite>`.

### Testing
- `<framework, fixtures, factories>`. Run **one file** at a time; `<required env var>` always.
- **Never** run the suite against a database holding data you did not create; read
  `testing.notes` in `.claude/workflow.json`.

### Lessons from code review
- Unenforced "keep in sync" invariants are findings — ask for a shared helper.
- Two names for the same entity in one scope after a round trip means one is stale.
- A test that stays green when the behaviour breaks proves nothing.
