# AGENTS.md

## Overview

This repository is a small Playwright workspace used for browser-driven investigation, UI debugging, and workflow testing.

## Shared Agent Policy

- Prefer one persistent browser owner for an entire debugging session.
- If the task spans multiple agent/tool calls, preserve browser state explicitly instead of assuming the environment will keep in-memory objects alive.
- If the user wants to watch the browser, run in non-headless mode.
- If the user asks for normal browsing behavior, use user-style actions only: open, goto, click, fill, type, select, check, uncheck, hover, snapshot, screenshot, and tab navigation.
- After navigation, modal changes, or failed interactions, refresh the page snapshot before reusing element refs.

## Playwright Session Guidance

For Playwright CLI session reuse, keep all of these stable across follow-up calls:

- `PLAYWRIGHT_DAEMON_SESSION_DIR`
- working directory
- CLI entrypoint
- session name

Recommended daemon directory:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon"
```

Recommended CLI entrypoint:

```bash
node node_modules/playwright-core/lib/tools/cli-client/cli.js
```

Launch once:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
open https://example.com --browser=chrome --headed
```

Reuse later:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot
```

## Repo Notes

- Browser/session artifacts are intentionally ignored in `.gitignore`:
  - `.playwright-cli`
  - `.pw-daemon`
  - `test-results`
  - `playwright-report`

- Existing Playwright tests live under `tests/`.
- The only agent-specific implementation currently kept in-repo is the Codex skill at `.codex/skills/playwright-cli-session/`.
