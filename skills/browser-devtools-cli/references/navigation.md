# Navigation Tools

Browser navigation commands for page control.

## go-to

Navigate to a URL.

```bash
browser-devtools-cli navigation go-to --url <url> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--url` | string | Yes | - | URL to navigate to (include scheme: `http://`, `https://`) |
| `--timeout` | number | No | `0` | Max operation time in ms (`0` = no timeout) |
| `--wait-until` | enum | No | `load` | When to consider navigation complete |
| `--wait-for-navigation` | boolean | No | `true` | Wait for navigation then network idle before snapshot/screenshot |
| `--wait-for-timeout-ms` | number | No | `30000` | Timeout for navigation and network idle wait (ms) |
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after navigation |
| `--snapshot-options` | object | No | - | JSON: `{"interactiveOnly":bool,"cursorInteractive":bool}` — snapshot ref options (same as a11y) |
| `--include-screenshot` | boolean | No | `false` | Take a screenshot after navigation (saved to disk; path in `screenshotFilePath`) |
| `--screenshot-options` | object | No | - | JSON: `outputPath`, `name`, `fullPage`, `type`, `annotate`, `includeBase64` |

**wait-until values:**
- `load` - Wait for `load` event
- `domcontentloaded` - Wait for `DOMContentLoaded` event
- `networkidle` - Wait for no network connections for 500ms (discouraged for testing)
- `commit` - Wait for network response and document loading started

**Examples:**

```bash
# Basic navigation
browser-devtools-cli navigation go-to --url "https://example.com"

# With timeout
browser-devtools-cli navigation go-to --url "https://example.com" --timeout 10000

# Wait for DOM ready
browser-devtools-cli navigation go-to --url "https://example.com" --wait-until domcontentloaded

# JSON output for parsing
browser-devtools-cli --json navigation go-to --url "https://example.com"
```

**Output (JSON):** When `--include-snapshot` is true (default), `output` (ARIA tree text) and `refs` (e1, e2, ...) are also returned. When `--include-screenshot` is true, `screenshotFilePath` is returned (screenshot saved to disk; use `screenshotOptions.includeBase64` only when the file cannot be read from the path).

```json
{
  "url": "https://example.com/",
  "status": 200,
  "statusText": "OK",
  "ok": true,
  "output": "...",
  "refs": { "e1": { "role": "button", "name": "Submit", "selector": "..." }, ... }
}
```

---

## go-back-or-forward

Navigate backward or forward in browser history.

```bash
browser-devtools-cli navigation go-back-or-forward --direction <back|forward> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--direction` | enum | Yes | - | `back` or `forward` |
| `--timeout` | number | No | `0` | Max operation time in ms |
| `--wait-until` | enum | No | `load` | When to consider navigation complete |
| `--wait-for-navigation` | boolean | No | `true` | Wait for navigation then network idle before snapshot/screenshot |
| `--wait-for-timeout-ms` | number | No | `30000` | Timeout for navigation and network idle wait (ms) |
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after navigation |
| `--snapshot-options` | object | No | - | JSON: `{"interactiveOnly":bool,"cursorInteractive":bool}` |
| `--include-screenshot` | boolean | No | `false` | Take a screenshot after navigation |
| `--screenshot-options` | object | No | - | JSON: same as go-to |

**Examples:**

```bash
browser-devtools-cli navigation go-back-or-forward --direction back
browser-devtools-cli navigation go-back-or-forward --direction forward
browser-devtools-cli navigation go-back-or-forward --direction back --wait-until domcontentloaded
```

---

## reload

Reload the current page. By default returns ARIA snapshot with refs (same as go-to).

```bash
browser-devtools-cli navigation reload [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--timeout` | number | No | `0` | Max operation time in ms |
| `--wait-until` | enum | No | `load` | When to consider navigation complete |
| `--wait-for-navigation` | boolean | No | `true` | Wait for reload then network idle before snapshot/screenshot |
| `--wait-for-timeout-ms` | number | No | `30000` | Timeout for reload and network idle wait (ms) |
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after reload |
| `--snapshot-options` | object | No | - | JSON: `{"interactiveOnly":bool,"cursorInteractive":bool}` |
| `--include-screenshot` | boolean | No | `false` | Take a screenshot after reload |
| `--screenshot-options` | object | No | - | JSON: same as go-to |

**Examples:**

```bash
# Simple reload
browser-devtools-cli navigation reload

# Reload and wait for network idle
browser-devtools-cli navigation reload --wait-until networkidle
```

## Workflow Example

```bash
SESSION="--session-id nav-test"

# Navigate to page
browser-devtools-cli $SESSION navigation go-to --url "https://example.com"

# Click a link (handled by interaction tools)
browser-devtools-cli $SESSION interaction click --selector "a.next-page"

# Go back
browser-devtools-cli $SESSION navigation go-back-or-forward --direction back

# Go forward
browser-devtools-cli $SESSION navigation go-back-or-forward --direction forward

# Reload
browser-devtools-cli $SESSION navigation reload
```
