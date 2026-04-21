---
name: playwright-cli-session
description: Keep a Playwright CLI browser session alive across multiple tool calls by launching a detached Playwright CLI daemon once, pinning a stable repo-local PLAYWRIGHT_DAEMON_SESSION_DIR, and then reusing the same workdir and session name for later click, fill, type, snapshot, screenshot, and tab commands. Use when an agent must browse visibly or headlessly through Playwright CLI/MCP, especially when the user wants normal user-style interactions instead of ad hoc scripts or run-code snippets.
---

# Playwright Cli Session

## Overview

Use the Playwright CLI daemon model, not a one-off shell process, to preserve browser state across multiple command invocations.
Launch the browser once with `open`, keep the daemon metadata in a stable repo-local directory, and reuse that same session for all later commands.

## Use This Workflow

- Prefer this skill when the task is website interaction through Playwright CLI and the browser must survive across separate tool calls.
- Prefer this skill when the user wants to watch the browser live with `--headed`.
- Prefer this skill when the user explicitly wants user-style actions only.
- Do not use plain Playwright scripts, `run-code`, or `eval` when the user asked for normal browsing behavior only.

## Core Rules

1. Use a stable daemon directory inside the repo, for example `"$PWD/.pw-daemon"`.
2. Keep the same `workdir` for every Playwright CLI call in the session.
3. Keep the same session name for every call. Default is `default`; do not switch it accidentally.
4. Launch with `open` exactly once per session, then reuse the same browser with later CLI commands.
5. Refresh the page snapshot after navigation, modal changes, or any failed action. Element refs are not stable forever.
6. If the user asked for user-like browsing, limit yourself to commands such as `open`, `goto`, `click`, `fill`, `type`, `select`, `check`, `uncheck`, `hover`, `snapshot`, `screenshot`, `tab-list`, `tab-new`, and `tab-select`.

## Why This Works

`open` starts a detached Playwright CLI daemon and writes a session file under the daemon session directory. Later Playwright CLI commands read that session file, recover the socket path, and reconnect to the same browser.

Do not try to preserve a long-lived shell process. Preserve the daemon session instead.

## Quick Start

First confirm a local CLI client exists. Prefer the direct local entrypoint:

```bash
node node_modules/playwright-core/lib/tools/cli-client/cli.js
```

If that path is unavailable, inspect the local Playwright install before choosing a fallback. Do not assume the global environment is configured.

Launch a visible browser session:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
open https://example.com --browser=chrome --headed
```

Launch a headless session:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
open https://example.com --browser=chrome
```

Then reuse the same session:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot

PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js click e10
```

## Standard Workflow

### 1. Bootstrap

- Choose one stable daemon directory in the repo.
- Choose headed or headless mode based on the task.
- Run `open` once.
- Immediately run `snapshot` to confirm the session is reachable.

### 2. Clear Blockers First

- Handle cookie dialogs, consent modals, interstitials, and region selectors before attempting search or product actions.
- If a click is intercepted by a backdrop or modal, take a fresh snapshot and target the blocking UI first.

### 3. Drive the Site Like a User

- Use `click` on visible controls.
- Use `fill` for specific editable fields and `type` when keyboard behavior matters.
- Use `snapshot` frequently enough to keep refs current, especially after navigation or modal changes.
- Use `screenshot` when the visual state matters more than the accessibility snapshot.

### 4. Preserve Session Reuse

For every follow-up command, reuse all of these:

- The same `PLAYWRIGHT_DAEMON_SESSION_DIR`
- The same `workdir`
- The same CLI entrypoint
- The same session name

If any of these change, reconnect may fail or a new session may be created.

### 5. Recover Cleanly

If the session is gone, reopen once using the same daemon directory and continue from there. Do not start inventing new session names or mixing multiple daemon directories unless the task explicitly requires separate browsers.

## Guardrails

- Do not assume a displayed Playwright code snippet means a fresh browser was launched. The CLI often prints generated code for a single action even when the same session is being reused.
- Do not reuse stale refs after page transitions.
- Do not mix direct Playwright scripts with Playwright CLI daemon calls in the same browsing task unless the user explicitly allows that.
- Do not let shell variable expansion remove the `$` in prompts or examples that need a literal skill name.

## Read These References

- Read [references/command-patterns.md](references/command-patterns.md) for reusable command templates and example flows.
- Read [references/recovery.md](references/recovery.md) when session reuse fails, refs go stale, or the environment blocks daemon/socket access.
