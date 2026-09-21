---
name: testing-standards
description: Test strategy rules for plans, implementation and review — proportional testing by risk, the regression-test rule for bug fixes, how to run tests safely (single file, required env), and what a test must assert. Policy comes from testing.policy in the project config.
---

# Testing standards

## Policy (from config)

| `testing.policy` | Meaning |
| --- | --- |
| `tdd` | Write the failing test first, then the code. Use `superpowers:test-driven-development` if installed. |
| `tests-with-features` | New behaviour ships with tests in the same PR. |
| `tests-on-request` | Tests are written when a human asks — except the regression rule below. |

**Regression rule** (`testing.regressionTestForBugFixes`, default true): a fix for a reported
defect ships with an automated test that **fails against the pre-fix code and passes against the
fix**. The plan states the failure scenario reproduced and the test file. Feature work writes
"N/A — feature work". This is the standing exception to "no tests unless asked".

## Test strategy in a plan

Proportional to risk. Name, per change:

- What is verified automatically (unit / integration / e2e) and the file.
- What is verified manually and the exact steps (viewport, account state, data).
- What is *not* verified and why that is acceptable.

## Writing tests

- Assert specific values and states, not "no exception".
- Cover: happy path, invalid input, not-found/unauthorised, failure of a dependency, and — for
  anything with retries or concurrency — the double-run case.
- Mock external services at the boundary; never the unit under test.
- A test that keeps passing when the behaviour it claims to pin is broken is a finding.
- Use existing fixtures/factories; do not build parallel test infrastructure.

## Running tests safely

- Prefer `commands.testSingleFile` during implementation; run `commands.test` before review.
- Read `testing.notes` first — it holds the dangerous facts (required env vars, databases a
  suite may drop, suites too slow to run whole). Ask before the first run of a session if the
  notes mention a shared or non-disposable database.
- A flaky result (fails then passes with no code change) is reported as flaky, not fixed
  silently and not ignored.
