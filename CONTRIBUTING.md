# Contributing

## What belongs here

- A command that at least two teams would run the same way, with every project fact in config.
- A skill that removes a rule from two or more commands, or a rule teams keep re-explaining.
- An agent only when an *independent context* is the point (review, challenge), not for
  convenience.
- Templates and examples that a new repository can copy without editing project names out.

What does not: product/domain knowledge, locale tooling, design-system generators, anything
with a Jira key or a person's login in it. Those live in the project's own `.claude/`.

## Process

1. Open an issue describing the repeated chore or the rule, with evidence (which repos, how
   often).
2. Branch, make the change, keep the file-size and one-rule-one-place conventions in
   `CLAUDE.md`.
3. Run it against a real ticket in a real repository via a local marketplace; include a short
   transcript summary in the PR.
4. Update `docs/Command-Catalog.md` / `docs/Skills-Catalog.md`, `CHANGELOG.md`, and the
   `version` fields in `.claude-plugin/plugin.json` and `marketplace.json` (semver: patch for
   wording, minor for a new command/skill/config key, major for a changed config shape).
5. Review by someone from a *different* team than the author — the point of the repo is
   portability.

## Sending lessons back

If your team's `ai-failures.md` has an entry that would apply anywhere (tooling misuse, a
verification habit), turn it into a sentence in the matching skill and open a PR. Keep the
ticket reference out; keep the lesson.
