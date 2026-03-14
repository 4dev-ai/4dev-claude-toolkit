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

### agent-device

Mobile automation for iOS simulators/devices and Android emulators/devices using [agent-device](https://github.com/callstackincubator/agent-device) by Callstack. Navigate apps, take snapshots/screenshots, tap, type, scroll, debug crashes, and more.

**Triggers**: "validate my changes", "check the simulator", "test this feature", "verify the UI", "navigate the app", "tap on", "take a screenshot"

**Requirements**:
- iOS: macOS with Xcode and simulators
- Android: Android SDK with emulator
- agent-device runs via npx, no additional setup needed

---

### playwright-interactive

Persistent browser and Electron interaction through Playwright for fast iterative UI debugging. Supports desktop, mobile, and Electron sessions with functional and visual QA checklists.

**Triggers**: "test the UI", "debug the browser", "check the page", "take a screenshot", "run visual QA", "test in browser"

To add the Playwright MCP server:
```bash
claude mcp add playwright -- npx @playwright/mcp@latest
```

Project setup:
```bash
pnpm add -D playwright
npx playwright install chromium
# For Electron apps:
# pnpm add -D electron
```

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
