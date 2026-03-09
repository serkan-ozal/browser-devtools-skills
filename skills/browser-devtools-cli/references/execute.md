# Execute Tool

Batch-execute multiple tool calls in a single request via custom JavaScript. Available in both **CLI** and **MCP**. Reduces round-trips and token usage. On the browser platform the VM receives `page` (Playwright Page); use `page.evaluate()` or the Playwright API, or `await callTool(name, input, returnOutput?)` to invoke other tools.

## execute

Run JavaScript in the session VM. Code is the body only (no async wrapper). Use `await callTool(name, input, returnOutput?)` to call other tools; use `page` (browser) for Playwright or `page.evaluate()`.

```bash
browser-devtools-cli run execute --code "<javascript>" [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--code` | string | Yes | - | JavaScript code (body only; do not wrap in async function). Use `await callTool(name, input, returnOutput?)` and/or `page` (browser). |
| `--timeout-ms` | number | No | `30000` | Wall-clock timeout for the entire execution in ms. Max: 120000. |

**Description:**

- Code runs in an async context; use `await` and `return` directly.
- Max 50 tool calls per execution; fail-fast on first error with `failedTool` in the response.

**Bindings (available inside your code):**

| Binding | Description |
|--------|-------------|
| **`page`** | **(Browser platform only.)** Playwright [Page](https://playwright.dev/docs/api/class-page) instance for the current session. Use for navigation, DOM, or in-page script. All async methods must be called with `await` (e.g. `await page.title()`, `await page.goto(url)`, `await page.evaluate(() => ...)`). |
| **`callTool(name, input, returnOutput?)`** | Invoke any registered MCP tool from inside execute. **Always use `await`** — it returns a Promise. See below. |

**`callTool(name, input, returnOutput?)` — arguments:**

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Tool name with underscore (e.g. `'navigation_go-to'`, `'content_take-screenshot'`, `'a11y_take-aria-snapshot'`). |
| `input` | object | Yes | Tool input: key-value map matching the tool’s parameters (camelCase, e.g. `{ url: 'https://example.com' }`, `{ selector: 'e1', value: 'text' }`). |
| `returnOutput` | boolean | No | Default `false`. When `true`, the tool’s output is also appended to the response `toolOutputs` array (and still returned from `callTool` for in-code use). |

- **Usage:** `await callTool('navigation_go-to', { url: 'https://example.com' });` or `const out = await callTool('content_get-as-text', {}, true);` — on failure `callTool` throws; response includes `failedTool` with the tool name and error.
- **`page` (browser):** e.g. `await page.title()`, `await page.evaluate(() => document.title)`, `await page.locator('button').click()`, `await page.goto(url)`. Not available on Node platform.

**Examples:**

```bash
# Return page title
browser-devtools-cli run execute --code "return await page.title();"

# Batch: snapshot then screenshot (returnOutput=true to get results in response)
browser-devtools-cli run execute --code "await callTool('a11y_take-aria-snapshot', {}, true); await callTool('content_take-screenshot', {}, true);"

# Custom timeout
browser-devtools-cli run execute --code "await page.evaluate(() => 1+1);" --timeout-ms 10000

# JSON output
browser-devtools-cli --json run execute --code "return await page.title();"
```

**Output (JSON):**

Response includes `toolOutputs` (where `callTool` was used with `returnOutput: true`), `logs` (console.log/warn/error), `result` (return value), and on error `error` and optionally `failedTool`.
