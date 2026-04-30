---
name: dialect
description: "Use when the user wants to change Claude's writing style or dialect. Activated via /dialect <name>, /dialect list, or /dialect off."
---

# Dialect

Switch Claude's writing style to a fun dialect.

## Usage

- `/dialect <name>` - Activate a dialect
- `/dialect list` - Show available dialects
- `/dialect off` - Deactivate the current dialect

## Handling Commands

### `/dialect list`

Scan the `${CLAUDE_PLUGIN_ROOT}/dialects/` directory for `.md` files. Skip `_template.md`. For each file, read the frontmatter `name` and `description` fields. Present them as a formatted list:

**Available dialects:**
- **name** - description

Also mention: "You can add custom dialects by dropping a `.md` file into the plugin's `dialects/` directory. See `_template.md` for the format."

### `/dialect <name>`

1. Look for `${CLAUDE_PLUGIN_ROOT}/dialects/<name>.md`
2. If the file does not exist, respond: "That dialect doesn't exist! Run `/dialect list` to see what's available."
3. If the file exists:
   a. Read the entire file
   b. Write the dialect name to `~/.claude/plugins/data/dialect/active-dialect.txt` (create the directory if needed)
   c. Present the **Full Patterns** section to establish the writing style
   d. Tell Claude: "From now on in this conversation, write all responses following these patterns. Stay in character at all times. Here are the rules:"
   e. Present the **Quick Reference** rules as the condensed instruction set
   f. Confirm to the user which dialect was activated

### `/dialect off`

1. Delete `~/.claude/plugins/data/dialect/active-dialect.txt` if it exists
2. Tell Claude: "Stop using any dialect. Return to your normal writing style."
3. Confirm to the user that the dialect has been deactivated.

## Important

- Only one dialect can be active at a time. Activating a new dialect replaces the previous one.
- The dialect applies to the current conversation only and does not persist across conversations.
- The reinforcement hook will re-inject the Quick Reference rules on each user message to maintain consistency.
