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
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after navigation |
| `--snapshot-interactive-only` | boolean | No | - | Only interactive elements get refs in snapshot |
| `--snapshot-cursor-interactive` | boolean | No | - | Include cursor:pointer/onclick elements in snapshot refs |

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

**Output (JSON):** When `--include-snapshot` is true (default), `output` (ARIA tree text) and `refs` (e1, e2, ...) are also returned.

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
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after navigation |
| `--snapshot-interactive-only` | boolean | No | - | Only interactive elements get refs in snapshot |
| `--snapshot-cursor-interactive` | boolean | No | - | Include cursor:pointer/onclick elements in snapshot refs |

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
| `--include-snapshot` | boolean | No | `true` | Return ARIA snapshot with refs after reload |
| `--snapshot-interactive-only` | boolean | No | - | Only interactive elements get refs |
| `--snapshot-cursor-interactive` | boolean | No | - | Include cursor:pointer/onclick in refs |

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
