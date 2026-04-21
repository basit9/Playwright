# Recovery

Use this checklist when the browser seems to disappear between calls.

## Root Cause Checklist

Verify all of these first:

1. Same `PLAYWRIGHT_DAEMON_SESSION_DIR`
2. Same `workdir`
3. Same CLI entrypoint
4. Same session name
5. Browser was started with `open`, not a plain Playwright script

If any one of these changed, reconnect may fail.

## Common Failures

### `EPERM: operation not permitted, mkdir '~/Library/Caches/ms-playwright/daemon'`

Cause:

- The environment blocks the default daemon cache location.

Fix:

- Pin the daemon directory inside the repo:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon"
```

### `Browser 'default' is not open`

Cause:

- The daemon exited
- The session directory changed
- The command ran from a different workspace or with a different session lookup context

Fix:

1. Confirm the daemon session files still exist under `.pw-daemon`
2. Re-run `open` once with the same daemon directory
3. Continue with the same session afterwards

### `listen EPERM` or local socket/connect permission errors

Cause:

- The environment blocks local daemon/socket operations

Fix:

- Use the same command path but request the permission level needed by the environment
- Keep the daemon directory stable after the permission change

### `Element is not an input` or a ref points to the wrong thing

Cause:

- The page rerendered and your old snapshot refs became stale
- A modal changed the accessible tree and shifted refs

Fix:

1. Take a fresh `snapshot`
2. Re-read the new refs
3. Retry with the new target

### `intercepts pointer events`

Cause:

- A cookie banner, consent dialog, or overlay is blocking the target

Fix:

1. Snapshot the current page
2. Identify the overlay controls
3. Dismiss or accept the blocker
4. Snapshot again before resuming

## Behavioral Guidance

- Reopening once is normal when the daemon genuinely died.
- Reopening before every action means the session reuse pattern is broken.
- The goal is not to keep a terminal process alive; the goal is to keep the Playwright CLI daemon alive.

## Success Signals

You are reusing the same session if later commands can:

- Run without another `open`
- Interact with the existing page state
- Keep tabs, cookies, and prior navigation history

You are probably not reusing the same session if:

- Every call starts from the homepage
- The site loses cookies or prior state
- The CLI repeatedly reports the browser is not open
