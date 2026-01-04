# 4dev Claude Toolkit

Development toolkit for Claude Code by [4dev.ai](https://4dev.ai).

## Features

### Tmux Integration
- **Auto-detect services** from `package.json` scripts
- **Start dev environment** in tmux with split panes
- **Monitor logs** from running services
- **Send commands** to specific panes
- **Diagnose errors** by checking service output

## Commands

| Command | Description |
|---------|-------------|
| `/4dev:startmux` | Start development services in tmux |

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

## Installation

```bash
claude --plugin-dir /path/to/4dev-claude-toolkit
```

## Requirements

- tmux installed and available in PATH
- Project with `package.json` (or agent will ask how to start)

## Roadmap

Future additions to the toolkit:
- Deployment workflows
- Testing utilities
- CI/CD integration

## License

MIT
