---
name: tmux-commands
description: This skill should be used when the user asks to "check tmux logs", "send command to tmux", "restart a service", "check what's running", "view pane output", "manage tmux session", or mentions tmux panes, dev services, or monitoring running processes. Provides tmux command patterns for development workflow management.
version: 0.1.0
---

# Tmux Commands for Development

Interact with tmux sessions to manage development services, monitor logs, and send commands to running processes.

## Core Concepts

- **Session**: Named container for windows/panes (typically one per project)
- **Window**: Named tab within a session (typically "dev" for development)
- **Pane**: Split section running a command (numbered from 0)
- **Target format**: `SESSION:WINDOW.PANE` (e.g., `myproject:dev.0`)

## Session Management

### Create session with services

```bash
# Kill existing session if present
tmux kill-session -t [PROJECT_NAME] 2>/dev/null

# Create session with primary service
tmux new-session -d -s "[PROJECT_NAME]" -n "dev" -c "$PWD" "[PRIMARY_CMD]"

# Add secondary service (horizontal split)
tmux split-window -h -t "[PROJECT_NAME]:dev" -c "$PWD" "[SECONDARY_CMD]"
```

### Three services (horizontal layout)

```bash
tmux new-session -d -s "[PROJECT_NAME]" -n "dev" -c "$PWD" "[CMD_1]"
tmux split-window -h -t "[PROJECT_NAME]:dev" -c "$PWD" "[CMD_2]"
tmux split-window -h -t "[PROJECT_NAME]:dev" -c "$PWD" "[CMD_3]"
tmux select-layout -t "[PROJECT_NAME]:dev" even-horizontal
```

### Session lifecycle

```bash
# List all sessions
tmux ls

# List panes in window
tmux list-panes -t "[PROJECT_NAME]:dev"

# Attach to session (for user)
tmux attach -t [PROJECT_NAME]

# Kill entire session
tmux kill-session -t [PROJECT_NAME]
```

## Pane Operations

### Check logs from pane

Capture recent output from a specific pane:

```bash
# Last 50 lines
tmux capture-pane -t "[PROJECT_NAME]:dev.[PANE_NUM]" -p -S -50

# Last 100 lines (for more context)
tmux capture-pane -t "[PROJECT_NAME]:dev.[PANE_NUM]" -p -S -100

# Full scrollback
tmux capture-pane -t "[PROJECT_NAME]:dev.[PANE_NUM]" -p -S -
```

### Send command to pane

Execute command in a running pane:

```bash
tmux send-keys -t "[PROJECT_NAME]:dev.[PANE_NUM]" "[COMMAND]" Enter
```

Common uses:
- Restart process: Send `Ctrl+C` then start command
- Clear terminal: Send `clear`
- Run additional command in existing pane

### Restart specific pane

```bash
# Kill and recreate pane
tmux kill-pane -t "[PROJECT_NAME]:dev.[PANE_NUM]"
tmux split-window -h -t "[PROJECT_NAME]:dev" -c "$PWD" "[CMD]"
tmux select-layout -t "[PROJECT_NAME]:dev" even-horizontal
```

### Send interrupt (Ctrl+C)

```bash
tmux send-keys -t "[PROJECT_NAME]:dev.[PANE_NUM]" C-c
```

## Layout Options

```bash
# Side by side
tmux select-layout -t "[PROJECT_NAME]:dev" even-horizontal

# Stacked vertically
tmux select-layout -t "[PROJECT_NAME]:dev" even-vertical

# Main pane left, others stacked right
tmux select-layout -t "[PROJECT_NAME]:dev" main-vertical

# Resize pane (positive = right/down, negative = left/up)
tmux resize-pane -t "[PROJECT_NAME]:dev.0" -R 20
```

## Flags Reference

| Flag | Meaning |
|------|---------|
| `-d` | Detached (don't attach) |
| `-h` | Horizontal split (side by side) |
| `-v` | Vertical split (top/bottom) |
| `-t` | Target pane/window/session |
| `-c` | Working directory |
| `-n` | Window name |
| `-s` | Session name |
| `-p` | Print to stdout (capture-pane) |
| `-S` | Start line for capture (-50 = last 50 lines) |

## Pane Numbering

- Panes numbered from `.0` (leftmost/top)
- After splits, new pane gets next number
- Use `tmux list-panes -t "[PROJECT_NAME]:dev"` to see current layout

## Error Suppression

```bash
# Suppress "session not found" errors
tmux kill-session -t [PROJECT_NAME] 2>/dev/null

# Check if session exists
tmux has-session -t [PROJECT_NAME] 2>/dev/null && echo "exists"
```

## Workflow Patterns

### Monitor for errors

1. Capture pane output: `tmux capture-pane -t "project:dev.0" -p -S -50`
2. Look for error patterns in output
3. If error found, capture more context with `-S -100`

### Restart crashed service

1. Send interrupt: `tmux send-keys -t "project:dev.0" C-c`
2. Wait briefly
3. Send start command: `tmux send-keys -t "project:dev.0" "pnpm dev" Enter`

### Check all panes quickly

```bash
for i in 0 1 2; do
  echo "=== Pane $i ==="
  tmux capture-pane -t "project:dev.$i" -p -S -20
done
```

## Project Name Detection

Derive session name from:
1. Git repository name: `basename $(git rev-parse --show-toplevel)`
2. Current directory: `basename $PWD`
3. Use kebab-case, lowercase

## Package.json Script Detection

Common dev scripts to look for:
- `dev`, `start`, `serve` - Main development server
- `dev:api`, `dev:server` - Backend service
- `dev:web`, `dev:client`, `dev:frontend` - Frontend service
- `watch`, `build:watch` - File watchers

Read scripts with: `cat package.json | jq -r '.scripts | keys[]'`
