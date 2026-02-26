# Node Run Commands

Execute JavaScript in the connected Node.js process. Requires `debug connect` first.

## js-in-node

Run JavaScript in the process context via CDP Runtime.evaluate. Has access to `process`, `require`, global, and Node APIs. Use `return` to pass a value back. Async supported. Return value must be JSON-serializable.

**MCP parameters:** `script`, `timeoutMs` (optional, default 5000, range 100–30000).

```bash
# Basic
node-devtools-cli run js-in-node --script "process.memoryUsage()"
node-devtools-cli run js-in-node --script "require('os').loadavg()"

# With return (JSON for parsing)
node-devtools-cli --json run js-in-node --script "return process.memoryUsage()"

# Async
node-devtools-cli --json run js-in-node --script "return await fetch('http://localhost:3000/health').then(r => r.json())"

# Timeout (ms)
node-devtools-cli run js-in-node --script "heavyWork()" --timeout-ms 10000
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--script` | string | Yes | - | JavaScript to run. Use `return` for result; async/await supported. |
| `--timeout-ms` | number | No | `5000` | Max evaluation time in ms (100–30000). |

**Output (JSON):** `result` — the evaluation result (primitives, arrays, or objects; must be JSON-serializable).
