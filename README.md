# 4dev Claude Toolkit

Development toolkit for Claude Code by [4dev.ai](https://github.com/4dev-ai/4dev-claude-toolkit).

## Skills

### tmux-commands

Tmux command reference for development workflow management. Provides patterns for session management, pane operations, log monitoring, and service control.

**Triggers**: "check tmux logs", "send command to tmux", "restart a service", "check what's running", "view pane output"

**Requirements**:
- tmux installed and available in PATH
- Project with `package.json` (or agent will ask how to start)

---

### ios-validation (expo-validation)

Validates Expo app implementations in the iOS Simulator after code changes. Captures screenshots, inspects the accessibility tree, and simulates user interactions.

**Triggers**: "validate my changes", "check the simulator", "does it look right?", "test this feature", "verify the UI"

**Requirements**:
- macOS with Xcode and iOS simulators installed
- ios-simulator-mcp server configured
- Facebook IDB tool (required by ios-simulator-mcp)
- Expo app running in simulator (Expo Go or dev build)

To add ios-simulator-mcp:
```bash
claude mcp add ios-simulator npx ios-simulator-mcp
```

---

### playwright-interactive

Persistent browser and Electron interaction through Playwright for fast iterative UI debugging. Supports desktop, mobile, and Electron sessions with functional and visual QA checklists.

**Triggers**: "test the UI", "debug the browser", "check the page", "take a screenshot", "run visual QA", "test in browser"

**Requirements**:
- Playwright installed in the project (`pnpm add -D playwright`)
- Browser binaries installed (`npx playwright install chromium`)
- For Electron: `pnpm add -D electron`

---

## Other Components

| Type | Name | Description |
|------|------|-------------|
| Command | `/4dev:startmux` | Start development services in tmux |
| Agent | `tmux-monitor` | Diagnoses errors from tmux service logs |

## Installation

### From GitHub (recommended)

First, add the repo as a marketplace:

```bash
claude plugin marketplace add github:4dev-ai/4dev-claude-toolkit
```

Then install the plugin:

```bash
claude plugin install 4dev
```

### For a single session (local directory)

```bash
claude --plugin-dir /path/to/4dev-claude-toolkit
```

## License

MIT
