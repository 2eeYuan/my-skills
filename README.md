# my-skills

A fast catalog plugin for [Claude Code](https://claude.ai/code) that lists all installed skills, plugins, and commands in seconds.

## What it does

Run `/my-skills` to instantly see:

- **All installed skills** from every marketplace (with descriptions)
- **All plugin commands** (slash commands)
- **All background plugins** (LSP servers, output styles, etc.)
- **All custom commands** (user-global and project-level)

## Performance

| Metric | Before | After |
|--------|--------|-------|
| Scan time | ~4 minutes | ~30 seconds |
| Tool calls | ~15 sequential | 1 bash call |
| Skills found | 106 | 129+ |

## Installation

### Option 1: Add as a marketplace (recommended)

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "my-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/2eeYuan/my-skills.git",
        "ref": "main"
      }
    }
  },
  "enabledPlugins": {
    "my-skills@my-skills": true
  }
}
```

### Option 2: Copy as a custom command

Copy `skills/my-skills/SKILL.md` to `~/.claude/commands/my-skills.md`.

## How it works

The skill runs a single bash script that:

1. Scans all marketplace directories for SKILL.md files
2. Scans all plugin directories for commands and skills
3. Falls back to marketplace.json for plugins without plugin.json
4. Scans user and project custom commands
5. Outputs structured data for Claude to format into a readable table

## Supported marketplace structures

- `marketplace/skills/skill-name/SKILL.md` (anthropics-skills style)
- `marketplace/plugin-name/skills/skill-name/SKILL.md` (pm-skills style)
- `marketplace/plugins/plugin-name/skills/skill-name/SKILL.md` (claude-plugins-official style)

## License

MIT
