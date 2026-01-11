---
name: expo-validation
description: Use when user asks to "validate my changes", "check the simulator", "test this feature", "verify the UI", "does it look right", "check if it works", or after implementing UI changes that need visual/interaction verification. Validates Expo app implementations in iOS simulator.
version: 0.2.0
---

# Expo App Validation

Validate Expo app implementations in iOS simulator after making code changes.

## When to Use

Proactively validate after:
- Implementing a user-requested feature or change
- Fixing a bug that affects UI or behavior
- User asks "does it work?" or "check the simulator"
- Completing a visual component implementation

## Workflow

```
Code Change → Hot Reload → Validate → Report → Iterate
```

The app hot-reloads automatically when code changes. Focus on validation, not installation.

## Quick Validation

### 1. Check Current State

Capture what's on screen:
- MCP Tool: `mcp__plugin_ios_simulator__ui_view`
- Returns compressed screenshot for quick viewing

### 2. Understand UI Structure

Get accessibility tree with element positions:
- MCP Tool: `mcp__plugin_ios_simulator__ui_describe_all`
- Returns: element types, labels, coordinates, interaction states

### 3. Test Interactions

Tap elements:
- MCP Tool: `mcp__plugin_ios_simulator__ui_tap`
- Parameters: `x`, `y` coordinates (get from ui_describe_all)

Type text:
- MCP Tool: `mcp__plugin_ios_simulator__ui_type`
- Parameter: `text` to enter

Swipe/scroll:
- MCP Tool: `mcp__plugin_ios_simulator__ui_swipe`
- Parameters: `x_start`, `y_start`, `x_end`, `y_end`

## Validation Patterns

### After Implementing a Feature

1. `ui_view()` - Capture current screen
2. Compare with user's requirements
3. Report what matches and what needs adjustment
4. If needed, iterate on implementation

### After UI Changes

1. `ui_view()` - See visual result
2. `ui_describe_all()` - Verify element structure
3. Check: layout, spacing, text content, colors
4. Report discrepancies with fix suggestions

### Testing User Flows

1. Identify flow steps from requirements
2. For each step:
   - Perform interaction (tap/type/swipe)
   - `ui_view()` - Capture result
   - Verify expected outcome
3. Report flow status

## Integration with Tmux

Check Expo dev server status if app seems unresponsive:

```bash
tmux capture-pane -t "[PROJECT]:dev.[EXPO_PANE]" -p -S -30
```

Look for:
- Build errors
- Metro bundler issues
- Hot reload failures

## Reporting Findings

After validation, report:

**What works:**
- Features that match requirements
- Interactions that behave correctly

**Issues found:**
- Specific problems with location/description
- Suggested code fixes

**Next steps:**
- Fixes to implement
- Follow-up validations needed

## MCP Tool Reference

| Tool | Purpose |
|------|---------|
| `ui_view` | Quick screenshot |
| `ui_describe_all` | Full accessibility tree |
| `ui_describe_point` | Element at specific coordinates |
| `ui_tap` | Simulate tap |
| `ui_type` | Enter text |
| `ui_swipe` | Scroll or swipe gestures |
| `screenshot` | High-quality PNG to file |
| `record_video` / `stop_recording` | Capture flow videos |

## Coordinates

- Origin (0, 0) at top-left
- X increases right, Y increases down
- Get element bounds from `ui_describe_all`
- Tap center: (x + width/2, y + height/2)

## Troubleshooting

| Issue | Check |
|-------|-------|
| No response to taps | Element coordinates correct? Element enabled? |
| Screen not updating | Tmux logs for Metro/build errors |
| Wrong screen shown | Navigation state, app may need manual reset |
| Text not appearing | Field focused first with ui_tap? |
