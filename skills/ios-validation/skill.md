---
name: expo-validation
description: Use when user asks to "validate my changes", "check the simulator", "test this feature", "verify the UI", "does it look right", "check if it works", or after implementing UI changes that need visual/interaction verification. Validates Expo app implementations in iOS/Android simulator using agent-device.
version: 0.3.0
---

# Expo App Validation

Validate Expo app implementations in iOS/Android simulator after making code changes using [agent-device](https://github.com/callstackincubator/agent-device).

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

Capture a screenshot and accessibility snapshot:
```bash
npx agent-device snapshot -i
```
Returns the accessibility tree with a screenshot.

### 2. Understand UI Structure

Get accessibility tree with element positions and refs:
```bash
npx agent-device snapshot
```
Returns: element types, labels, coordinates, `@ref` identifiers for targeting.

### 3. Test Interactions

Tap elements (by coordinates, @ref, or selector):
```bash
npx agent-device click @ref
npx agent-device click 200 400
```

Type text in focused field:
```bash
npx agent-device type "hello world"
```

Tap then type (fill):
```bash
npx agent-device fill 200 400 "hello world"
npx agent-device fill @ref "hello world"
```

Swipe/scroll:
```bash
npx agent-device swipe 200 600 200 200
npx agent-device scroll down 0.5
```

Long press:
```bash
npx agent-device longpress 200 400 1000
```

## Validation Patterns

### After Implementing a Feature

1. `npx agent-device snapshot -i` — capture current screen with screenshot
2. Compare with user's requirements
3. Report what matches and what needs adjustment
4. If needed, iterate on implementation

### After UI Changes

1. `npx agent-device snapshot -i` — see visual result
2. `npx agent-device snapshot` — verify element structure
3. Check: layout, spacing, text content, colors
4. Report discrepancies with fix suggestions

### Testing User Flows

1. Identify flow steps from requirements
2. For each step:
   - Perform interaction (click/type/swipe)
   - `npx agent-device snapshot -i` — capture result
   - Verify expected outcome
3. Report flow status

### Waiting for Elements

```bash
npx agent-device wait text "Welcome"
npx agent-device wait @ref
npx agent-device wait 2000
```

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

## Command Reference

| Command | Purpose |
|---------|---------|
| `snapshot -i` | Screenshot + accessibility tree |
| `snapshot` | Accessibility tree only |
| `click <x y\|@ref\|selector>` | Tap element |
| `type <text>` | Type in focused field |
| `fill <target> <text>` | Tap then type |
| `swipe <x1> <y1> <x2> <y2>` | Swipe gesture |
| `scroll <direction> [amount]` | Scroll (0-1 amount) |
| `longpress <x> <y> [ms]` | Long press |
| `wait text <text>` | Wait for text to appear |
| `wait <ms>` | Wait duration |
| `back` | Navigate back |
| `home` | Go to home screen |
| `alert get\|accept\|dismiss` | Handle alerts |
| `apps` | List installed apps |
| `open <app>` | Launch app |
| `close [app]` | Close app |

## Coordinates

- Origin (0, 0) at top-left
- X increases right, Y increases down
- Get element positions from `snapshot`
- Use `@ref` identifiers from snapshot for reliable targeting

## Troubleshooting

| Issue | Check |
|-------|-------|
| No response to taps | Element coordinates correct? Use `@ref` instead |
| Screen not updating | Tmux logs for Metro/build errors |
| Wrong screen shown | Navigation state, app may need manual reset |
| Text not appearing | Field focused first with click? |
| Device not found | Run `npx agent-device devices` to list available |
