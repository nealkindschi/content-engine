# agent-browser command reference

Full command listing for agent-browser, grouped by purpose. Commands most
relevant to the content-engine pipeline are marked with ★.

## Navigation

| Command | Description |
|---------|-------------|
| ★ `open <url>` | Launch browser and navigate to URL (aliases: `goto`, `navigate`) |
| `open` | Launch browser with no navigation (stays on about:blank) |
| `back` | Go back |
| `forward` | Go forward |
| `reload` | Reload page |

## Reading content ★

| Command | Description |
|---------|-------------|
| ★ `snapshot` | Full accessibility tree |
| ★ `snapshot -i` | Interactive elements only |
| ★ `snapshot -i -c` | Compact output |
| ★ `snapshot -i -d <n>` | Limit depth |
| ★ `snapshot -s <sel>` | Scope to CSS selector |
| ★ `get text <sel>` | Visible text content |
| `get html <sel>` | innerHTML |
| `get title` | Page title |
| `get url` | Current URL |
| `get value <sel>` | Input value |
| `get attr <sel> <attr>` | Attribute value |
| `get count <sel>` | Count matching elements |

## Screenshots

| Command | Description |
|---------|-------------|
| `screenshot [path]` | Take screenshot |
| `screenshot --full` | Full page screenshot |
| `screenshot --annotate` | Numbered labels + refs legend |

## Waiting ★

| Command | Description |
|---------|-------------|
| ★ `wait <selector>` | Wait for element to be visible |
| ★ `wait <ms>` | Wait milliseconds |
| ★ `wait --load networkidle` | Wait for network idle |
| `wait --text "..."` | Wait for text to appear |
| `wait --url "**/pattern"` | Wait for URL pattern |
| `wait --fn "js condition"` | Wait for JS condition |

## Interacting

| Command | Description |
|---------|-------------|
| `click <sel>` | Click element |
| `click <sel> --new-tab` | Open link in new tab |
| `dblclick <sel>` | Double-click |
| `focus <sel>` | Focus element |
| `type <sel> <text>` | Type into element |
| `fill <sel> <text>` | Clear and fill |
| `press <key>` | Press key (Enter, Tab, etc.) |
| `hover <sel>` | Hover element |
| `select <sel> <val>` | Select dropdown option |
| `check <sel>` | Check checkbox |
| `uncheck <sel>` | Uncheck checkbox |
| `scroll <dir> [px]` | Scroll (up/down/left/right) |
| `scrollintoview <sel>` | Scroll element into view |

## Tabs ★

| Command | Description |
|---------|-------------|
| ★ `tab` | List tabs |
| ★ `tab new [url]` | New tab |
| ★ `tab <id>` | Switch to tab |
| ★ `tab close [id]` | Close tab |

## Find (semantic locators)

| Command | Description |
|---------|-------------|
| `find role <role> <action>` | By ARIA role |
| `find text <text> <action>` | By text content |
| `find label <label> <action>` | By label |
| `find placeholder <ph> <action>` | By placeholder |
| `find first <sel> <action>` | First match |
| `find nth <n> <sel> <action>` | Nth match |

## Cookies & storage

| Command | Description |
|---------|-------------|
| `cookies` | Get all cookies |
| `cookies clear` | Clear cookies |
| `storage local` | Get localStorage |

## Session management

| Command | Description |
|---------|-------------|
| `--session <name>` | Isolated browser session |
| `--session-name <name>` | Auto-save/restore session |
| `--state <path>` | Load saved auth state |
| `state save <path>` | Save auth state |
| `state load <path>` | Load auth state |

## Setup & diagnostics

| Command | Description |
|---------|-------------|
| ★ `install` | Download Chrome for Testing |
| ★ `doctor` | Diagnose installation |
| `doctor --fix` | Repair installation |
| `upgrade` | Upgrade to latest version |

## Cleanup

| Command | Description |
|---------|-------------|
| ★ `close` | Close browser |
| ★ `close --all` | Close all sessions |

## Global flags

| Flag | Description |
|------|-------------|
| `--headed` | Show browser window |
| `--json` | JSON output |
| `--session <name>` | Isolated session |
| `--cdp <port>` | Connect via CDP |
| `--auto-connect` | Auto-discover running Chrome |
| `--executable-path <path>` | Custom browser executable |
| `--proxy <url>` | Proxy server |
| `--profile <name\|path>` | Chrome profile |
| `--headers <json>` | HTTP headers |
| `--user-agent <ua>` | Custom User-Agent |
| `--allowed-domains <list>` | Domain allowlist |

## Environment variables

| Variable | Description |
|----------|-------------|
| `AGENT_BROWSER_SESSION` | Default session name |
| `AGENT_BROWSER_SESSION_NAME` | Auto-save session name |
| `AGENT_BROWSER_PROFILE` | Chrome profile name/path |
| `AGENT_BROWSER_EXECUTABLE_PATH` | Custom browser path |
| `AGENT_BROWSER_PROXY` | Proxy URL |
| `AGENT_BROWSER_HEADED` | Show browser window |
| `AGENT_BROWSER_DEFAULT_TIMEOUT` | Operation timeout in ms |
| `AGENT_BROWSER_ALLOWED_DOMAINS` | Domain allowlist |
