# 4dev Claude Toolkit

Development toolkit for Claude Code by [4dev.ai](https://github.com/4dev-ai/4dev-claude-toolkit).

## Features

### Tmux Integration
- **Auto-detect services** from `package.json` scripts
- **Start dev environment** in tmux with split panes
- **Monitor logs** from running services
- **Send commands** to specific panes
- **Diagnose errors** by checking service output

### Expo Validation
- **Validate implementations** after code changes
- **Visual verification** via iOS Simulator MCP
- **Interaction testing** with tap, swipe, and input simulation
- **User flow validation** for complete journeys
- **Tmux integration** to check dev server status

## Components

| Type | Name | Description |
|------|------|-------------|
| Command | `/4dev:startmux` | Start development services in tmux |
| Agent | `tmux-monitor` | Diagnoses errors from tmux service logs |
| Skill | `tmux-commands` | Tmux command reference for dev workflows |
| Skill | `expo-validation` | Expo app validation in iOS simulator |

## Usage

### Start Development Session

```
/4dev:startmux
```

Detects your project's start scripts and launches them in a tmux session with split panes. Automatically begins monitoring for issues.

### During Development

The tmux-monitor agent automatically:
- Checks relevant pane logs when you mention errors
- Diagnoses issues based on service output
- Suggests fixes or restarts services as needed

### Expo App Validation

After implementing features, validate with phrases like:
- "validate my changes"
- "check the simulator"
- "does it look right?"
- "test this feature"

The skill provides patterns for:
- Visual verification of code changes
- Interaction testing (taps, swipes, text input)
- User flow validation
- Integration with tmux to check dev server status

**Prerequisites**: Configure ios-simulator-mcp server in your Claude settings

## Installation

### From GitHub (recommended)

```bash
claude plugins add github:4dev-ai/4dev-claude-toolkit
```

### From local directory

```bash
claude plugins add /path/to/4dev-claude-toolkit
```

## Requirements

### Tmux Features
- tmux installed and available in PATH
- Project with `package.json` (or agent will ask how to start)

### Expo Validation Features
- macOS with Xcode and iOS simulators installed
- ios-simulator-mcp server configured
- Facebook IDB tool (required by ios-simulator-mcp)
- Expo app running in simulator (Expo Go or dev build)

To add ios-simulator-mcp:
```bash
claude mcp add ios-simulator npx ios-simulator-mcp
```

## Roadmap

Future additions to the toolkit:
- Deployment workflows
- Testing utilities
- CI/CD integration

## License

MIT
