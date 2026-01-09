---
name: build
description: Build a React or Next.js web application - automatically initializes new projects or continues development on existing ones
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep", "TodoWrite", "AskUserQuestion", "WebSearch", "WebFetch", "Task"]
---

# Build Web Application

This command automatically detects whether you're starting a new project or continuing an existing one.

## Detection Logic

**Check for the progress file:** `.claude/progress/claude-progress.txt`

- **If the file exists** → This is an existing project. Launch the **webapp-coding** agent.
- **If the file does NOT exist** → This is a new project. Launch the **webapp-initializer** agent.

## Instructions

1. First, check if `.claude/progress/claude-progress.txt` exists in the current working directory
2. Based on the result, launch the appropriate agent:

### For New Projects (no progress file)

Launch the **initializer** agent to:
- Gather project requirements
- Create project structure with Docker Compose
- Generate feature specifications
- Set up deployment infrastructure
- Initialize progress tracking

### For Existing Projects (progress file exists)

Launch the **coding** agent to:
- Read the progress file to understand current state
- Review feature specifications
- Plan the session with focused tasks
- Implement features incrementally
- Update progress log for continuity

## Usage

```
/build
```

With optional direction:
```
/build Work on the authentication feature
/build Fix the form validation bug
/build Set up the user dashboard
```

For new projects, the initializer will ask questions about requirements.
For existing projects, any additional context you provide will guide the coding session.

---

**Detect project state and launch the appropriate agent now.**
