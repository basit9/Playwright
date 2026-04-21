# Command Patterns

Use these patterns verbatim, then substitute the URL, refs, and arguments you need.

## Assumptions

- Current working directory is the repo root.
- Local Playwright is already installed in `node_modules`.
- The preferred CLI client is:

```bash
node node_modules/playwright-core/lib/tools/cli-client/cli.js
```

## Stable Prefix

Reuse the same daemon directory on every command:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon"
```

## Open Once

Visible browser:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
open https://example.com --browser=chrome --headed
```

Headless browser:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
open https://example.com --browser=chrome
```

## Inspect the Page

Full snapshot:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot
```

Partial snapshot:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot e101
```

Element screenshot:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js \
screenshot e101 --filename .playwright-cli/section.png
```

## User-Style Actions

Click a visible control:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js click e10
```

Fill a search box:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js fill e59 "iPhone Air"
```

Type and submit:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js type "Magi65 Aluminum" --submit
```

Navigate directly:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js goto https://example.com/products
```

## Realistic Flow

Cookie banner, search, inspect:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js click e19

PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js fill e59 "iPhone Air"

PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js click e60

PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot
```

Open a result and inspect variants:

```bash
PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js click e3706

PLAYWRIGHT_DAEMON_SESSION_DIR="$PWD/.pw-daemon" \
node node_modules/playwright-core/lib/tools/cli-client/cli.js snapshot
```

## Session Inspection

If you need to confirm that the daemon wrote session metadata, inspect the daemon directory:

```bash
find .pw-daemon -maxdepth 3 -type f | sort
```

Typical files:

- `.../default.session`
- `.../default.err`

The session file is the key artifact that lets later CLI calls reconnect to the same browser.

## When to Refresh Refs

Take a fresh snapshot when:

- The page navigated
- A modal opened or closed
- A click was intercepted
- The target ref unexpectedly resolves to the wrong element
- You selected a product variant and the page rerendered
