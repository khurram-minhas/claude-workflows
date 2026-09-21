---
name: figma-design
description: When and how to pull design context from Figma during ticket analysis and planning (get_design_context, get_screenshot, get_variable_defs), how to record what was used in the plan, and what to do without the Figma MCP. Optional integration; enabled via figma.enabled.
---

# Figma design context

Applies when `figma.enabled` is true **and** the ticket or the user references a Figma URL. If
the Figma MCP tools (`mcp__figma__*`) are not present, say so once and ask for screenshots
instead; never describe a design you have not seen.

## What to fetch

Extract `fileKey` and `node-id` from the URL (`figma.com/design/<fileKey>/...?node-id=<id>`).

| Need | Tool | Use it for |
| --- | --- | --- |
| Layout, spacing, text, component names | `get_design_context` | The primary read. Ask for the specific node, not the page. |
| A picture to compare against the rendered result | `get_screenshot` | Attach the comparison to self-review for UI tickets. |
| Colours, typography, spacing tokens | `get_variable_defs` | Map to the project's existing design tokens; never hardcode a hex that a token already provides. |
| Where a component already exists in code | `get_code_connect_map` | Reuse before building. |

Load the Figma plugin's own skill (`/figma-use` or the `skill://figma/...` resource) before any
write to Figma. Shared commands never write to Figma.

## Rules

- Trust the swatch hex over any text label that disagrees with it.
- Map every design token to an existing project token first; list genuinely new tokens
  explicitly in the plan under "Architecture impact → Styling".
- Record the Figma node links used in the plan header (`**Design:** <links>`), so reviewers can
  open the same frame.
- Design context informs the plan; it does not replace acceptance criteria from the ticket. When
  the design and the ticket disagree, raise it as an open question — do not pick one silently.
- For UI tickets, self-review compares a screenshot of the running app against
  `get_screenshot` output and notes differences rather than asserting "matches Figma".
