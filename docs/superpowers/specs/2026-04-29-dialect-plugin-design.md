# Dialect Plugin Design Spec

## Overview

A Claude Code plugin that lets users switch Claude's writing style between fun dialects via a single `/dialect` skill. Ships with five built-in dialects and supports user-created custom dialects via markdown files.

## Plugin Structure

```
claude-dialect-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── dialect/
│       └── SKILL.md
├── dialects/
│   ├── _template.md
│   ├── frat-boy.md
│   ├── stoner.md
│   ├── valley-girl.md
│   ├── southern-drawl.md
│   └── yosemite-sam.md
├── hooks/
│   ├── hooks.json
│   └── reinforce-dialect.cmd
├── package.json
├── README.md
└── LICENSE
```

## Skill: `/dialect`

Single skill with three commands:

- **`/dialect <name>`** - Activates a dialect. Reads the matching `.md` file from `dialects/`, loads the Full Patterns section to set the writing style, and extracts the Quick Reference for reinforcement.
- **`/dialect list`** - Lists all available dialects by scanning the `dialects/` folder (skips `_template.md`). Shows name and description from each file's frontmatter.
- **`/dialect off`** - Deactivates the current dialect. Claude returns to normal writing style.

### Frontmatter

```yaml
---
name: dialect
description: "Use when the user wants to change Claude's writing style or dialect. Activated via /dialect <name>, /dialect list, or /dialect off."
---
```

### Behavior

- Dialect is active for the duration of the conversation only. Does not persist across conversations.
- `/dialect nonexistent` returns a helpful error message and suggests running `/dialect list`.

## Dialect File Format

Each dialect is a markdown file in `dialects/` with this structure:

```markdown
---
name: dialect-name
description: Short description of the dialect
---

## Quick Reference
<!-- 10-15 concise rules. Re-injected by the reinforcement hook. -->
1. Rule one
2. Rule two
...

## Full Patterns
<!-- Rich, detailed writing guide. Loaded on first activation. -->
### Section
...
```

### Constraints

- Quick Reference: max ~15 rules. Must stay concise to minimize token cost on reinforcement.
- Full Patterns: no hard limit, but shorter is better for context window preservation.
- `_template.md` is a blank dialect file with placeholder text and instructions for creating custom dialects. Prefixed with underscore so it sorts to the top and the skill skips it.

### Custom Dialects

Users add custom dialects by dropping a `.md` file into the `dialects/` directory following the same format. Custom dialects are immediately available via `/dialect list` and `/dialect <name>` with no additional configuration.

## Reinforcement Hook

### Problem

As conversations get long, Claude loses adherence to dialect patterns due to attention decay and context compression.

### Solution

A hook registered in `hooks/hooks.json` that fires on the `UserPromptSubmit` event (when the user sends a message). This means the dialect rules are re-injected right before Claude processes each new message - fresh in context every time.

When a dialect is active:

1. Reads the active dialect's Quick Reference section from a cached state file
2. Returns it as additional context that gets injected into the prompt

Lives at `hooks/reinforce-dialect.cmd`.

State tracking: when `/dialect <name>` activates a dialect, it writes the active dialect name to a state file (e.g., `~/.claude/plugins/data/dialect/active-dialect.txt`). The hook reads this file to know which dialect (if any) is active and which Quick Reference to inject. `/dialect off` deletes the state file.

### Design Decisions

- Fires on every user prompt submission. Simple, and the Quick Reference is small enough that token cost is negligible.
- If token usage becomes a problem, can be adjusted to fire less frequently (e.g., every N messages).

## Built-in Dialects

Five dialects ship with v1.0.0:

1. **frat-boy** - Talks like a fraternity bro (research doc already exists)
2. **stoner** - Laid-back, spacey, philosophical vibes
3. **valley-girl** - Like, totally SoCal speak
4. **southern-drawl** - Southern charm and colloquialisms
5. **yosemite-sam** - Rootin' tootin' Wild West bluster

## Plugin Metadata

`plugin.json`:
```json
{
  "name": "dialect",
  "description": "Switch Claude's writing style between fun dialects",
  "version": "1.0.0"
}
```

## Success Criteria

- `/dialect frat-boy` activates the dialect and Claude writes in that style
- `/dialect list` shows all available dialects (built-in + custom)
- `/dialect off` deactivates the dialect
- Reinforcement hook keeps the dialect consistent over long conversations
- Dropping a new `.md` file in `dialects/` makes it immediately available
- `/dialect nonexistent` gives a helpful error message
