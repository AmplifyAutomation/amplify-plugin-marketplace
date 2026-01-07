# amplify-plugin-marketplace

A marketplace for defining and managing custom Claude Code plugins.

## Structure

```
amplify-plugin-marketplace/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (required)
├── commands/                 # Slash commands (.md files)
├── agents/                   # Subagent definitions (.md files)
├── skills/                   # Agent skills (subdirectories with SKILL.md)
├── hooks/                    # Event handlers (hooks.json)
├── scripts/                  # Helper scripts and utilities
├── marketplace.json          # Plugin registry
├── .gitignore
└── README.md
```

## Marketplace Registry

The `marketplace.json` file is the central registry for all plugins in the marketplace.

### Adding a Plugin to the Registry

Add a new entry to the `plugins` array in `marketplace.json`:

```json
{
  "name": "my-plugin",
  "displayName": "My Plugin",
  "description": "What this plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Your Name",
    "email": "you@example.com",
    "url": "https://github.com/username"
  },
  "repository": "https://github.com/username/my-plugin",
  "homepage": "https://example.com/docs",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "category": "utilities",
  "components": {
    "commands": 2,
    "agents": 1,
    "skills": 1,
    "hooks": 0,
    "mcp": 0
  },
  "compatibility": {
    "claudeCode": ">=1.0.0"
  },
  "featured": false,
  "verified": false
}
```

### Plugin Entry Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (kebab-case) |
| `displayName` | Yes | Human-readable name |
| `description` | Yes | Brief description of functionality |
| `version` | Yes | Semantic version (MAJOR.MINOR.PATCH) |
| `author` | Yes | Author info (name, email, url) |
| `repository` | No | Source code URL |
| `homepage` | No | Documentation URL |
| `license` | No | License identifier (MIT, Apache-2.0, etc.) |
| `keywords` | No | Search/filter keywords |
| `category` | No | Plugin category |
| `components` | No | Count of each component type |
| `compatibility` | No | Claude Code version requirements |
| `featured` | No | Highlight in marketplace |
| `verified` | No | Verified by maintainers |

## Adding Components

### Commands

Create `.md` files in the `commands/` directory:

```markdown
---
name: my-command
description: What this command does
allowed-tools: ["Read", "Write", "Bash"]
---

Instructions for Claude when this command is invoked...
```

Usage: `/amplify-plugin-marketplace:my-command`

### Agents

Create `.md` files in the `agents/` directory:

```markdown
---
name: my-agent
whenToUse: Description of when to trigger this agent
model: sonnet
tools: ["Read", "Glob", "Grep"]
---

System prompt for the agent...
```

### Skills

Create subdirectories in `skills/` with a `SKILL.md` file:

```
skills/
└── my-skill/
    ├── SKILL.md           # Required: skill definition
    ├── references/        # Optional: detailed docs
    └── examples/          # Optional: code examples
```

SKILL.md format:

```markdown
---
name: My Skill
description: When Claude should use this skill
---

Skill instructions and knowledge...
```

### Hooks

Create `hooks/hooks.json`:

```json
{
  "PreToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{
      "type": "command",
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
    }]
  }]
}
```

### MCP Servers

Create `.mcp.json` at plugin root:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"]
    }
  }
}
```

## Usage

### Local Development

```bash
claude --plugin-dir /path/to/amplify-plugin-marketplace
```

### Install as Plugin

Copy to your Claude Code plugins directory or use the marketplace installer.

## Best Practices

1. **Use `${CLAUDE_PLUGIN_ROOT}`** for all path references in hooks and MCP configs
2. **Kebab-case naming** for all files and directories
3. **Minimal manifest** - rely on auto-discovery for standard layouts
4. **Document components** - add clear descriptions in frontmatter
5. **Update marketplace.json** - keep the registry in sync with available plugins

## License

MIT
