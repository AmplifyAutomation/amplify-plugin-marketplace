# Amplify Plugin Marketplace

A curated collection of Claude Code plugins for accelerating development workflows.

## Installation

Add this marketplace to Claude Code:

```bash
claude --plugin-dir /path/to/amplify-plugin-marketplace
```

Or for permanent installation, add to your Claude Code settings.

## Available Plugins

### webapp-dev

A two-stage agent system for building production-ready React and Next.js applications with Docker Compose local development and cloud deployment to AWS EC2 or Digital Ocean Droplets.

#### Quick Start

```bash
/build
```

That's it. The `/build` command automatically detects your project state and runs the appropriate agent.

#### How It Works

The plugin uses two specialized agents:

| Agent | Triggered When | Purpose |
|-------|---------------|---------|
| **initializer** | No progress file exists | Sets up new projects with Docker, deployment scripts (AWS EC2 or Digital Ocean), and feature planning |
| **coding** | Progress file exists | Continues development, tracks progress, implements features incrementally |

**Detection logic:** Checks for `.claude/progress/claude-progress.txt`

#### Usage Examples

**Starting a new project:**
```bash
/build
```
The initializer agent will ask about your project requirements (including your preferred cloud provider), then create:
- Project structure with Docker Compose
- Feature specifications (`docs/FEATURES.md`)
- Deployment configuration for your chosen provider (AWS EC2 or Digital Ocean)
- Progress tracking system

**Continuing development:**
```bash
/build
/build Work on the authentication feature
/build Fix the form validation bug
```
The coding agent will:
1. Read your progress file to understand current state
2. Review feature specifications
3. Create a focused task list
4. Implement features incrementally
5. Update progress log for the next session

#### What Gets Created

```
your-project/
├── .claude/
│   └── progress/
│       └── claude-progress.txt    # Session continuity
├── docker/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   └── nginx.conf
├── scripts/
│   ├── init.sh
│   ├── dev.sh
│   └── deploy.sh
├── docs/
│   ├── FEATURES.md
│   ├── ARCHITECTURE.md
│   └── DEPLOYMENT.md
├── docker-compose.yml
├── docker-compose.prod.yml
└── .env.example
```

#### Included Skills

The plugin includes specialized knowledge for:
- **Next.js Patterns** - App Router, server components, data fetching
- **React Components** - Component architecture, hooks, state management
- **Supabase Integration** - Auth, database, real-time subscriptions
- **Tailwind UI** - Responsive design, component styling

## Repository Structure

```
amplify-plugin-marketplace/
├── .claude-plugin/
│   └── marketplace.json       # Plugin registry
├── webapp-dev/                # Web app development plugin
│   ├── agents/
│   │   ├── coding.md          # Incremental development agent
│   │   └── initializer.md     # Project setup agent
│   ├── commands/
│   │   └── build.md           # Unified /build command
│   ├── skills/                # Domain knowledge
│   └── scripts/templates/     # Project templates
├── CLAUDE.md                  # Claude Code instructions
└── README.md
```

## Creating Your Own Plugin

Each plugin follows this structure:

```
plugin-name/
├── .mcp.json              # MCP server configuration
├── commands/              # Slash commands (.md files)
├── agents/                # Subagent definitions (.md files)
├── skills/                # Agent skills (subdirectories with SKILL.md)
├── hooks/                 # Event handlers (hooks.json)
└── scripts/               # Helper scripts
```

### Adding to the Marketplace

1. Create your plugin directory
2. Add an entry to `.claude-plugin/marketplace.json`:

```json
{
  "name": "your-plugin",
  "displayName": "Your Plugin Name",
  "description": "What your plugin does",
  "source": "./your-plugin"
}
```

## License

MIT
