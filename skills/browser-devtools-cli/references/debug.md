# Debug Commands

Non-blocking debugging tools that capture snapshots without pausing execution.

## Probe Types

| Type | Description |
|------|-------------|
| `tracepoint` | Captures call stack, local variables, async traces |
| `logpoint` | Evaluates expressions (lightweight, no call stack) |
| `exceptionpoint` | Captures on exceptions |
| `dompoint` | Monitors DOM mutations |
| `netpoint` | Monitors network requests |

## Tracepoint Commands

### Put Tracepoint

```bash
browser-devtools-cli debug put-tracepoint \
  --url-pattern "app.js" \
  --line-number 42

# With condition
browser-devtools-cli debug put-tracepoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --condition "user.id === 123"

# With hit condition (every 10th hit)
browser-devtools-cli debug put-tracepoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --hit-condition "% 10 == 0"
```

### List/Remove/Clear Tracepoints

```bash
browser-devtools-cli debug list-tracepoints
browser-devtools-cli debug remove-tracepoint --id "tp_abc123"
browser-devtools-cli debug clear-tracepoints
```

### Get/Clear Tracepoint Snapshots

```bash
# Get all tracepoint snapshots
browser-devtools-cli --json debug get-tracepoint-snapshots

# Filter by specific tracepoint
browser-devtools-cli --json debug get-tracepoint-snapshots --probe-id "tp_abc123"

# Polling: get new snapshots since sequence 50
browser-devtools-cli --json debug get-tracepoint-snapshots --from-sequence 50

# Limit results
browser-devtools-cli --json debug get-tracepoint-snapshots --limit 10

# Clear snapshots
browser-devtools-cli debug clear-tracepoint-snapshots
```

## Logpoint Commands

### Put Logpoint

```bash
# Simple value
browser-devtools-cli debug put-logpoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --log-expression "user.name"

# Template string
browser-devtools-cli debug put-logpoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --log-expression '`User: ${user.name}, Age: ${user.age}`'

# Object
browser-devtools-cli debug put-logpoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --log-expression "{ user, timestamp: Date.now() }"
```

### List/Remove/Clear Logpoints

```bash
browser-devtools-cli debug list-logpoints
browser-devtools-cli debug remove-logpoint --id "lp_abc123"
browser-devtools-cli debug clear-logpoints
```

### Get/Clear Logpoint Snapshots

```bash
browser-devtools-cli --json debug get-logpoint-snapshots
browser-devtools-cli debug clear-logpoint-snapshots
```

## Exceptionpoint Commands

### Put Exceptionpoint

```bash
# Catch uncaught exceptions only
browser-devtools-cli debug put-exceptionpoint --state uncaught

# Catch all exceptions (including caught)
browser-devtools-cli debug put-exceptionpoint --state all

# Disable exception catching
browser-devtools-cli debug put-exceptionpoint --state none
```

### Get/Clear Exceptionpoint Snapshots

```bash
browser-devtools-cli --json debug get-exceptionpoint-snapshots
browser-devtools-cli debug clear-exceptionpoint-snapshots
```

## Dompoint Commands

### Put Dompoint

```bash
# Monitor subtree changes
browser-devtools-cli debug put-dompoint \
  --selector "#app" \
  --type subtree-modified

# Monitor attribute changes
browser-devtools-cli debug put-dompoint \
  --selector "button.submit" \
  --type attribute-modified

# Monitor specific attribute
browser-devtools-cli debug put-dompoint \
  --selector "input" \
  --type attribute-modified \
  --attribute-name "disabled"

# Monitor node removal
browser-devtools-cli debug put-dompoint \
  --selector ".modal" \
  --type node-removed
```

### List/Remove/Clear Dompoints

```bash
browser-devtools-cli debug list-dompoints
browser-devtools-cli debug remove-dompoint --id "dp_abc123"
browser-devtools-cli debug clear-dompoints
```

### Get/Clear Dompoint Snapshots

```bash
browser-devtools-cli --json debug get-dompoint-snapshots
browser-devtools-cli debug clear-dompoint-snapshots
```

## Netpoint Commands

### Put Netpoint

```bash
# Monitor all API requests
browser-devtools-cli debug put-netpoint --url-pattern "/api/*"

# Monitor specific endpoint
browser-devtools-cli debug put-netpoint --url-pattern "/api/users"

# Monitor only POST requests
browser-devtools-cli debug put-netpoint \
  --url-pattern "/api/*" \
  --method POST

# Capture on request (before sent)
browser-devtools-cli debug put-netpoint \
  --url-pattern "/api/*" \
  --timing request

# Capture on response (default)
browser-devtools-cli debug put-netpoint \
  --url-pattern "/api/*" \
  --timing response
```

### List/Remove/Clear Netpoints

```bash
browser-devtools-cli debug list-netpoints
browser-devtools-cli debug remove-netpoint --id "np_abc123"
browser-devtools-cli debug clear-netpoints
```

### Get/Clear Netpoint Snapshots

```bash
browser-devtools-cli --json debug get-netpoint-snapshots
browser-devtools-cli debug clear-netpoint-snapshots
```

## Watch Expressions

Watch expressions are evaluated at every tracepoint/exceptionpoint hit.

```bash
# Add watch expressions
browser-devtools-cli debug add-watch --expression "this"
browser-devtools-cli debug add-watch --expression "arguments"
browser-devtools-cli debug add-watch --expression "Object.keys(state)"

# List watches
browser-devtools-cli debug list-watches

# Remove specific watch
browser-devtools-cli debug remove-watch --id "w_abc123"

# Clear all watches
browser-devtools-cli debug clear-watches
```

## Debug Status

```bash
browser-devtools-cli --json debug status
```

Returns:
- `enabled`: Whether debugging is active
- `hasSourceMaps`: Whether source maps are loaded
- `exceptionBreakpoint`: Current state (none, uncaught, all)
- `tracepointCount`: Number of tracepoints
- `logpointCount`: Number of logpoints
- `dompointCount`: Number of DOM breakpoints
- `netpointCount`: Number of network breakpoints
- `watchExpressionCount`: Number of watch expressions
- `snapshotStats`: Snapshot statistics

## Snapshot Format

### Tracepoint Snapshot

```json
{
  "id": "snap_123",
  "probeId": "tp_456",
  "timestamp": 1706300000000,
  "sequenceNumber": 5,
  "url": "http://localhost:3000/bundle.js",
  "lineNumber": 42,
  "originalLocation": {
    "source": "src/app.ts",
    "line": 15,
    "column": 8
  },
  "callStack": [
    {
      "functionName": "handleClick",
      "url": "bundle.js",
      "lineNumber": 42,
      "scopes": [
        {
          "type": "local",
          "variables": [
            { "name": "event", "value": {...}, "type": "object" }
          ]
        }
      ],
      "originalLocation": { "source": "src/app.ts", "line": 15 }
    }
  ],
  "asyncStackTrace": {
    "segments": [
      { "description": "Promise.then", "callFrames": [...] }
    ]
  },
  "watchResults": {
    "this": { "className": "Button" },
    "arguments": [...]
  },
  "captureTimeMs": 12
}
```

### Logpoint Snapshot

```json
{
  "id": "snap_124",
  "probeId": "lp_789",
  "timestamp": 1706300000100,
  "sequenceNumber": 6,
  "url": "bundle.js",
  "lineNumber": 50,
  "originalLocation": { "source": "src/utils.ts", "line": 20 },
  "logResult": "User: john, Age: 25",
  "captureTimeMs": 2
}
```

## Examples

### Debug a Click Handler

```bash
SESSION="--session-id debug-test"

# Navigate
browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"

# Set tracepoint on click handler
browser-devtools-cli $SESSION debug put-tracepoint \
  --url-pattern "app.js" \
  --line-number 100

# Click the button (triggers tracepoint)
browser-devtools-cli $SESSION interaction click --selector "#submit-btn"

# Get snapshots
browser-devtools-cli $SESSION --json debug get-tracepoint-snapshots
```

### Monitor API Calls

```bash
# Set netpoint for API
browser-devtools-cli debug put-netpoint --url-pattern "/api/*"

# Navigate and interact
browser-devtools-cli navigation go-to --url "http://localhost:3000"
browser-devtools-cli interaction click --selector "#load-data"

# Check captured requests
browser-devtools-cli --json debug get-netpoint-snapshots
```

### Catch Exceptions

```bash
# Enable exception catching
browser-devtools-cli debug put-exceptionpoint --state uncaught

# Run actions that might throw
browser-devtools-cli navigation go-to --url "http://localhost:3000/buggy"

# Check for exceptions
browser-devtools-cli --json debug get-exceptionpoint-snapshots
```

### Debug with Source Maps

Tracepoints work with bundled code and automatically resolve to original source locations when source maps are available.

```bash
# Set tracepoint using original source path
browser-devtools-cli debug put-tracepoint \
  --url-pattern "src/components/Button.tsx" \
  --line-number 25

# Snapshots will include originalLocation with source file info
```
