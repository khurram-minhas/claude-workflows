# Installing

## For one engineer (user scope)

```bash
claude plugin marketplace add khurram-minhas/claude-workflows
claude plugin install arbisoft-workflows@arbisoft-claude-workflows
claude plugin list                      # should show arbisoft-workflows … enabled
```

`/plugin marketplace add …` is the same thing from the terminal CLI's prompt. The slash form is
not available inside the VS Code / JetBrains extensions; the `claude plugin …` shell form is.

The marketplace is cloned with your own git credentials (SSH key or `gh auth login`). If the
repository is private and background refreshes fail, run `gh auth setup-git` once, or set
`CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1` so the last good copy is kept.

## For a repository (every teammate, automatically)

Commit this to the repository's shared `.claude/settings.json` (not `settings.local.json`):

```json
{
  "extraKnownMarketplaces": {
    "arbisoft-claude-workflows": {
      "source": { "source": "github", "repo": "khurram-minhas/claude-workflows", "ref": "v0.1.0" }
    }
  },
  "enabledPlugins": {
    "arbisoft-workflows@arbisoft-claude-workflows": true
  }
}
```

When a teammate opens the repository and trusts the folder, Claude Code adds the marketplace
and enables the plugin — no commands to run. `ref` pins a tag so upstream changes do not alter
the team's workflow until someone bumps it deliberately. The read-only permission allowlist
from `templates/settings.json` can live in the same file.

Then run `/setup-workflow team` (or `solo`) once to create `.claude/workflow.json`.

## Updating

```bash
claude plugin marketplace update arbisoft-claude-workflows
```

Repositories that pin `ref` update by changing the tag in `settings.json`.

## Developing the plugin

```bash
git clone git@github.com:khurram-minhas/claude-workflows.git
claude plugin marketplace add ./claude-workflows        # loaded in place; edits apply on next session
claude plugin validate ./claude-workflows               # before every PR
```

If you had installed from GitHub first, remove that marketplace (`claude plugin marketplace
remove arbisoft-claude-workflows`) before adding the local one under the same name.

## Without plugin support

Copy `commands/`, `skills/` and `agents/` into the repository's `.claude/` directory. Commands
then appear un-namespaced (`/plan`), and updates are manual.
