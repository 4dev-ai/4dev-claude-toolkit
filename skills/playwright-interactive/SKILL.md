---
name: playwright-interactive
description: "Persistent browser and Electron interaction through Playwright for fast iterative UI debugging. Use when the user asks to 'test the UI', 'debug the browser', 'check the page', 'take a screenshot', 'run visual QA', 'test in browser', or mentions Playwright, browser testing, Electron debugging, or visual verification."
version: 0.1.0
---

# Playwright Interactive Skill

Use a persistent Playwright session to debug local web or Electron apps, run functional plus visual QA, and capture screenshots without restarting the whole toolchain.

Adapted from the [OpenAI Codex playwright-interactive skill](https://github.com/openai/skills/tree/main/skills/.curated/playwright-interactive) (Apache 2.0, Copyright Microsoft Corporation).

## Preconditions

- Playwright must be installed in the project workspace.
- Run setup from the same project directory you need to debug.
- All Playwright interactions run through `node -e` or inline Node.js scripts via Bash.
- A persistent Node.js REPL process can be managed in a tmux pane for session continuity.

## One-time setup

```bash
test -f package.json || pnpm init
pnpm add -D playwright
# Web-only, for headed Chromium or mobile emulation:
# npx playwright install chromium
# Electron-only:
# pnpm add -D electron
node -e "import('playwright').then(() => console.log('playwright import ok')).catch((error) => { console.error(error); process.exit(1); })"
```

If you switch to a different workspace later, repeat setup there.

## Core Workflow

1. Write a brief QA inventory before testing:
   - Build the inventory from three sources: the user's requested requirements, the user-visible features or behaviors you actually implemented, and the claims you expect to make in the final response.
   - Anything that appears in any of those three sources must map to at least one QA check before signoff.
   - List the user-visible claims you intend to sign off on.
   - List every meaningful user-facing control, mode switch, or implemented interactive behavior.
   - List the state changes or view changes each control or implemented behavior can cause.
   - Use this as the shared coverage list for both functional QA and visual QA.
   - For each claim or control-state pair, note the intended functional check, the specific state where the visual check must happen, and the evidence you expect to capture.
   - Add at least 2 exploratory or off-happy-path scenarios that could expose fragile behavior.
2. Start or confirm any required dev server (preferably in a tmux pane).
3. Launch Playwright in a persistent Node.js session or via scripts.
4. After each code change, reload for renderer-only changes or relaunch for main-process/startup changes.
5. Run functional QA with normal user input simulation.
6. Run a separate visual QA pass.
7. Verify viewport fit and capture screenshots.
8. Clean up the Playwright session only when the task is actually finished.

## Running Playwright via Bash

Since Claude Code uses Bash rather than a persistent JS REPL, use one of these approaches:

### Approach 1: Inline Node.js scripts

```bash
node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({ viewport: { width: 1600, height: 900 } });
  const page = await context.newPage();
  await page.goto('http://127.0.0.1:3000', { waitUntil: 'domcontentloaded' });
  console.log('Loaded:', await page.title());
  await page.screenshot({ path: '/tmp/screenshot.png', type: 'png' });
  console.log('Screenshot saved to /tmp/screenshot.png');
  await browser.close();
})();
"
```

Then read the screenshot with the Read tool to view it.

### Approach 2: Persistent REPL in tmux

For session continuity across multiple interactions, run a Node.js REPL in a tmux pane:

```bash
# Start a persistent node REPL in tmux
tmux new-session -d -s playwright -n repl "node --experimental-repl-await"

# Send commands to the REPL
tmux send-keys -t playwright:repl "const { chromium } = require('playwright')" Enter
tmux send-keys -t playwright:repl "const browser = await chromium.launch({ headless: true })" Enter
tmux send-keys -t playwright:repl "const context = await browser.newContext({ viewport: { width: 1600, height: 900 } })" Enter
tmux send-keys -t playwright:repl "const page = await context.newPage()" Enter
tmux send-keys -t playwright:repl "await page.goto('http://127.0.0.1:3000', { waitUntil: 'domcontentloaded' })" Enter

# Capture output
tmux capture-pane -t playwright:repl -p -S -20
```

### Approach 3: Script files for complex flows

For complex QA flows, write a temporary script:

```bash
cat > /tmp/pw-test.mjs << 'EOF'
import { chromium } from 'playwright';

const browser = await chromium.launch({ headless: true });
const context = await browser.newContext({
  viewport: { width: 1600, height: 900 },
});
const page = await context.newPage();

await page.goto('http://127.0.0.1:3000', { waitUntil: 'domcontentloaded' });
console.log('Loaded:', await page.title());

// Functional QA
await page.click('button#submit');
await page.waitForSelector('.result');
const text = await page.textContent('.result');
console.log('Result:', text);

// Screenshot
await page.screenshot({ path: '/tmp/screenshot.png' });
console.log('Screenshot saved');

await browser.close();
EOF

node /tmp/pw-test.mjs
```

## Choose Session Mode

- Use an explicit viewport for routine iteration, reproducible screenshots, and consistent layout checks. This is the default.
- Use native-window mode (`viewport: null`) for a separate headed pass when you need to validate launched window size or OS-level behavior.
- For Electron, assume native-window behavior. Electron launches with `noDefaultViewport`.
- Treat switching modes as a context reset.

## Desktop Web Context

```javascript
const browser = await chromium.launch({ headless: true });
const context = await browser.newContext({
  viewport: { width: 1600, height: 900 },
});
const page = await context.newPage();
await page.goto('http://127.0.0.1:3000', { waitUntil: 'domcontentloaded' });
```

## Mobile Web Context

```javascript
const context = await browser.newContext({
  viewport: { width: 390, height: 844 },
  isMobile: true,
  hasTouch: true,
});
const page = await context.newPage();
await page.goto('http://127.0.0.1:3000', { waitUntil: 'domcontentloaded' });
```

## Electron Session

```javascript
import { _electron as electron } from 'playwright';

const electronApp = await electron.launch({ args: ['.'] });
const appWindow = await electronApp.firstWindow();
console.log('Loaded:', await appWindow.title());
```

## Screenshots

Save screenshots to `/tmp/` and read them with the Read tool:

```javascript
// Full page
await page.screenshot({ path: '/tmp/screenshot.png' });

// Specific element
await page.locator('#my-element').screenshot({ path: '/tmp/element.png' });

// Clipped region
await page.screenshot({
  path: '/tmp/region.png',
  clip: { x: 0, y: 0, width: 800, height: 600 },
});
```

## Checklists

### Session Loop

- Launch Playwright (inline, tmux REPL, or script).
- Start the target runtime.
- Make the code change.
- Reload or relaunch using the correct path for that change.
- Update the shared QA inventory if exploration reveals additional items.
- Re-run functional QA.
- Re-run visual QA.
- Capture final screenshots.

### Reload Decision

- Renderer-only change: reload the existing page.
- Main-process, preload, or startup change: relaunch Electron.
- New uncertainty about process ownership: relaunch instead of guessing.

### Functional QA

- Use real user controls for signoff: keyboard, mouse, click, touch, or equivalent Playwright input APIs.
- Verify at least one end-to-end critical flow.
- Confirm the visible result of that flow, not just internal state.
- Work through the shared QA inventory rather than ad hoc spot checks.
- Cover every obvious visible control at least once before signoff.
- For reversible controls or stateful toggles, test the full cycle: initial state, changed state, and return to initial.
- After scripted checks pass, do a short exploratory pass for 30-90 seconds.
- `page.evaluate(...)` may inspect or stage state, but it does not count as signoff input.

### Visual QA

- Treat visual QA as separate from functional QA.
- Use the same shared QA inventory.
- Restate user-visible claims and verify each one explicitly.
- Inspect the initial viewport before scrolling.
- Confirm that the initial view visibly supports the interface's primary claims.
- Inspect all required visible regions, not just the main interaction surface.
- Look for clipping, overflow, distortion, layout imbalance, inconsistent spacing, alignment problems, illegible text, weak contrast, broken layering.
- Judge aesthetic quality as well as correctness.
- Prefer viewport screenshots for signoff. Use full-page captures only as secondary debugging artifacts.

### Viewport Fit Checks (Required)

Before signoff, verify viewport fit:

```javascript
console.log(await page.evaluate(() => ({
  innerWidth: window.innerWidth,
  innerHeight: window.innerHeight,
  clientWidth: document.documentElement.clientWidth,
  clientHeight: document.documentElement.clientHeight,
  scrollWidth: document.documentElement.scrollWidth,
  scrollHeight: document.documentElement.scrollHeight,
  canScrollX: document.documentElement.scrollWidth > document.documentElement.clientWidth,
  canScrollY: document.documentElement.scrollHeight > document.documentElement.clientHeight,
})));
```

- Define the intended initial view before signoff.
- Use screenshots as the primary evidence for fit.
- Signoff fails if any required visible region is clipped, cut off, or pushed outside the viewport.
- Check region bounds, not just document bounds.

### Signoff

- The functional path passed with normal user input.
- Coverage is explicit against the shared QA inventory.
- The visual QA pass covered the whole relevant interface.
- Each user-visible claim has a matching visual check and reviewed screenshot.
- The viewport-fit checks passed.
- The UI is not just functional; it is visually coherent.
- A short exploratory pass was completed for interactive products.
- Include a brief negative confirmation of the main defect classes you checked for and did not find.
- Cleanup was executed, or you intentionally kept the session alive for further work.

## Cleanup

```javascript
await browser.close();
// For Electron:
await electronApp.close();
```

Or if using tmux REPL:

```bash
tmux send-keys -t playwright:repl "await browser.close()" Enter
tmux kill-session -t playwright
```

## Common Failure Modes

- `Cannot find module 'playwright'`: run the one-time setup in the current workspace.
- Browser executable missing: run `npx playwright install chromium`.
- `page.goto: net::ERR_CONNECTION_REFUSED`: make sure the dev server is running, recheck the port, prefer `http://127.0.0.1:<port>`.
- `electron.launch` hangs or exits: verify the local `electron` dependency, confirm the `args` target, ensure any renderer dev server is already running.
- Headed mode not working: on remote/headless servers, use `headless: true` (the default).
