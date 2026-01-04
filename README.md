# claude-tmux

Claude Code plugin for interacting with tmux during development.

## Features

- **Auto-detect services** from `package.json` scripts
- **Start dev environment** in tmux with split panes
- **Monitor logs** from running services
- **Send commands** to specific panes
- **Diagnose errors** by checking service output

## Usage

### Start Development Session

```
/startmux
```

Detects your project's start scripts and launches them in a tmux session with split panes. Automatically begins monitoring for issues.

### During Development

The tmux-monitor agent automatically:
- Checks relevant pane logs when you mention errors
- Diagnoses issues based on service output
- Suggests fixes or restarts services as needed

## Installation

```bash
claude --plugin-dir /path/to/claude-tmux
```

## Requirements

- tmux installed and available in PATH
- Project with `package.json` (or agent will ask how to start)
