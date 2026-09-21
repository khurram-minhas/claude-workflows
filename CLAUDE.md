# arbisoft-claude-workflows — context for working on this repository

This repository *is* a Claude Code plugin. Files here are prompts and rules, not application
code. When editing:

- **No project facts.** No product names, Jira keys, custom-field ids, GitHub repos, logins,
  labels, Confluence ids, or people anywhere outside `examples/` and `docs/Xiangqi-*.md`.
  Anything that varies per project goes through `.claude/workflow.json`
  (`config/workflow.schema.json`).
- **One rule, one place.** Rules live in `skills/`; commands only sequence them and reference
  the skill by name. If you find yourself pasting a rule into a second command, move it to a
  skill.
- **Commands stop at gates.** Ticket choice, story points, plan sign-off, plan deviation, PR
  creation, thread resolution — a command that acts outwardly without a human yes is a bug.
- **Null means skip, not guess.** A command meeting an unconfigured key says what it skipped.
- **Keep files short.** Commands under ~80 lines, skills under ~120. Frontmatter:
  `description` (and `argument-hint`) on commands; `name` + `description` on skills;
  `name`, `description`, `model`, `disallowedTools` on agents.
- **Update the catalogs** (`docs/Command-Catalog.md`, `docs/Skills-Catalog.md`) and bump
  `version` in both `.claude-plugin/*.json` in the same change.
- Test a change by adding this directory as a local marketplace in a real repository and
  running the command against a real ticket; paste the transcript summary in the PR.
