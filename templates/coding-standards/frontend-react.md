# <Project> — Coding Standards (React frontend starter)

Derived from a production React 18 / Redux / styled-components codebase. Part 1 is
stack-agnostic. Part 2 is a **starting point** — delete what is not true here and add what is.

---

## Part 1 — Core AI engineering rules

1. **Never assume architecture** — inspect siblings (`saga`, `reducer`, `selectors`, `Wrapper`)
   and understand Redux vs Context ownership before proposing anything.
2. **Prefer existing project patterns** — existing utilities, hooks, sagas, styling conventions;
   no new abstractions unless asked.
3. **Hallucination prevention** — never invent endpoints, selectors, reducers, translation keys,
   props. State assumptions; ask.
4. **Performance-safe React** — no inline object/array/function literals in render, no broken
   memoisation, no layout-property animations.
5. **Concurrency in sagas** — explain cancellation (`takeLatest`), races, retries.
6. **Refactor rules** — behaviour first, minimal diff, incremental.
7. **Verification requirement** — lint and run before "done"; name edge cases and assumptions.
8. **Human decisions stay human** — options with a recommendation for anything product-shaped.

---

## Part 2 — This codebase (edit me)

### Structure
- Containers connect to the store; components are presentational.
- `containers/Feature/{index,actions,constants,reducer,saga,selectors,messages,Wrapper}.js`
- `components/Name/{index,Wrapper,messages}.js`

### Styled components
- One `Wrapper` export per file with nested class selectors — never multiple exports.
- Theme tokens only (`props.theme.<token>`), never hardcoded colours.
- Media queries via the project's `media` helpers; animate `transform`/`opacity` only.

### Redux / Saga
- Selectors with `createSelector`; never inline selectors in `mapStateToProps`.
- Action types namespaced `containers/Feature/ACTION`.
- `takeLatest` for fetches, `takeEvery` only when every event matters; handle success **and**
  failure; reducers pure, spread not mutate, handle reset on unmount.

### i18n
- No hardcoded user-facing strings; `defineMessages` + `formatMessage`; keys scoped
  `app.containers.Feature.key`; add to every locale file; no string concatenation.

### Performance
- Stable references from selectors; `React.memo`/`useMemo`/`useCallback` where a child is
  expensive or re-renders often — not everywhere by default.
- Socket/event listeners registered in `useEffect` with cleanup; events go through Redux.

### Accessibility
- Interactive elements keyboard-reachable; `alt` on images; correct ARIA roles.

### Lint and quality gate
- `<lint command>` — zero errors, zero new warnings; no `eslint-disable` without a reason.
- Import order: React → libraries → images → utils → components → containers → relative.
- Quality-gate rules that bite: duplicated literals (extract constants), cognitive complexity
  > 15 (split), unused variables, TODO comments, nested ifs.

### Testing
- `<test policy>`. Bug fixes ship a regression test. `<how to run one file>`.

### Lessons from code review
- No magic strings/numbers — use existing constants.
- Mirror components must mirror states (loading / error / empty) and messages.
- Search `components/` before building a new Button/Modal/Badge.
