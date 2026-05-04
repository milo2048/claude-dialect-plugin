# Dialect Plugin for Claude Code

Switch Claude's writing style between fun dialects.

## Installation

```bash
claude plugin install /path/to/claude-dialect-plugin --scope user
```

Or try it out for a single session without installing:

```bash
claude --plugin-dir /path/to/claude-dialect-plugin
```

## Usage

```
/dialect list          # See available dialects
/dialect frat-boy      # Activate a dialect
/dialect off           # Back to normal
```

## Built-in Dialects

- **frat-boy** - Talks like a fraternity bro
- **stoner** - Laid-back, spacey, philosophical
- **valley-girl** - Like, totally SoCal speak
- **southern-drawl** - Southern charm and folksy wisdom
- **yosemite-sam** - Rootin' tootin' Wild West bluster
- **cookie-monster** - Me-want-cookie grammar and OM NOM NOM energy

## Custom Dialects

Add your own dialect by dropping a `.md` file into the `dialects/` directory. Copy `dialects/_template.md` to get started.

Your dialect file needs:
- YAML frontmatter with `name` and `description`
- A `## Quick Reference` section (10-15 concise rules, used for reinforcement)
- A `## Full Patterns` section (detailed writing guide)

## How It Works

When you activate a dialect, the full writing patterns are loaded into the conversation. A reinforcement hook re-injects the Quick Reference rules on every message to keep Claude on track over long conversations.
