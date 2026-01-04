---
description: Start development process in Tmux
allowed-tools: Bash, Read
---

# Start Development Services in Tmux

Start and manage multiple development services in tmux panes with automatic monitoring.

## Steps

### 1. Detect Project Configuration

Determine the project name:
```bash
basename $(git rev-parse --show-toplevel 2>/dev/null || pwd)
```

Read package.json to find available scripts:
```bash
cat package.json 2>/dev/null | grep -E '"(dev|start|serve|watch)"' || echo "NO_PACKAGE_JSON"
```

### 2. Identify Start Commands

Look for these patterns in package.json scripts:
- `dev`, `start`, `serve` - Main development server
- `dev:api`, `dev:server`, `dev:backend` - Backend service
- `dev:web`, `dev:client`, `dev:frontend` - Frontend service
- `watch` - File watchers

If no package.json or unclear scripts:
- Ask user which commands to run
- Offer common options: `pnpm dev`, `pnpm start`, `npm run dev`

### 3. Create Tmux Session

Kill existing session if present, then create new session with detected services.

For single service:
```bash
tmux kill-session -t [PROJECT_NAME] 2>/dev/null
tmux new-session -d -s "[PROJECT_NAME]" -n "dev" -c "$PWD" "[CMD]"
```

For two services (e.g., api + web):
```bash
tmux kill-session -t [PROJECT_NAME] 2>/dev/null
tmux new-session -d -s "[PROJECT_NAME]" -n "dev" -c "$PWD" "[CMD_1]"
tmux split-window -h -t "[PROJECT_NAME]:dev" -c "$PWD" "[CMD_2]"
tmux select-layout -t "[PROJECT_NAME]:dev" even-horizontal
```

### 4. Confirm Setup

After creating session, list panes to confirm:
```bash
tmux list-panes -t "[PROJECT_NAME]:dev" -F "Pane #{pane_index}: #{pane_current_command}"
```

Report to user:
- Session name and how to attach (`tmux attach -t [PROJECT_NAME]`)
- Pane layout (which service is in which pane)
- Note that monitoring is now active

### 5. Initial Log Check

Capture initial output from each pane to verify services started:
```bash
tmux capture-pane -t "[PROJECT_NAME]:dev.0" -p -S -20
```

Check for:
- Startup messages
- Port bindings
- Any immediate errors

Report startup status to user.

## Monitoring Mode

After starting, enter continuous monitoring mode:
- When user mentions errors, check relevant pane logs
- Proactively diagnose issues from service output
- Suggest restarts or fixes as needed

Use the tmux-monitor agent for ongoing monitoring.

## Notes

- Use `pnpm` as default package manager
- Panes numbered from 0 (leftmost)
- `-h` creates horizontal split (side by side)
- `-d` keeps session detached
