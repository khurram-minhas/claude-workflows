# Example: Python service in the same team

Shows the `extends` layer: tracker ids, reviewer pool and AI weights come from
`examples/team-config/workflow.team.json`; this file only states what differs for the service —
a different PR title style, `ruff`, a test suite that must never be run whole
(`testing.notes` is read by `/implement` and `/self-review` before any test run), and plans
kept under `docs/plans/`.

`commands.test` is deliberately `null`: the full suite is not something a command should run
here. `testSingleFile` is the safe path.

Pair with `templates/coding-standards/backend-python.md`.
