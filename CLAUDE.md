# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin marketplace - a registry and framework for managing custom Claude Code plugins. It serves as both a template for plugin developers and a central marketplace for publishing plugins.

## Development

### Local Testing

```bash
claude --plugin-dir /path/to/amplify-plugin-marketplace
```

### No Build System

This is a configuration-based project. There are no build steps, tests, or linting commands. Plugins are defined through Markdown files with YAML frontmatter and JSON configuration files.

## Architecture

### Plugin Registry

The central registry is at `.claude-plugin/marketplace.json`. Each plugin entry requires:
- `name`: Unique kebab-case identifier
- `displayName`: Human-readable name
- `description`: Brief functionality description
- `source`: Relative path to plugin directory

### Plugin Structure

Each plugin directory follows this standard layout:
```
plugin-name/
├── .mcp.json         # MCP server configuration
├── commands/         # Slash commands (.md files)
├── agents/           # Subagent definitions (.md files)
├── skills/           # Agent skills (subdirectories with SKILL.md)
├── hooks/            # Event handlers (hooks.json)
└── scripts/          # Helper scripts
```

### Component Formats

**Commands** (`commands/*.md`):
```markdown
---
name: command-name
description: What this command does
allowed-tools: ["Read", "Write", "Bash"]
---
Instructions for Claude...
```

**Agents** (`agents/*.md`):
```markdown
---
name: agent-name
whenToUse: When to trigger this agent
model: sonnet
tools: ["Read", "Glob", "Grep"]
---
System prompt...
```

**Skills** (`skills/skill-name/SKILL.md`):
```markdown
---
name: Skill Name
description: When Claude should use this skill
---
Skill instructions...
```

**Hooks** (`hooks/hooks.json`):
```json
{
  "PreToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{"type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"}]
  }]
}
```

Hook events: `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`, `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreCompact`

**MCP Servers** (`.mcp.json`):
```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"]
    }
  }
}
```

## Key Conventions

- Use `${CLAUDE_PLUGIN_ROOT}` for all path references in hooks and MCP configs (auto-substituted at runtime)
- Kebab-case naming for all files and directories
- Components are auto-discovered based on directory structure
- Keep `marketplace.json` in sync with available plugins
