# Debug Commands

Non-blocking debugging tools that capture snapshots without pausing execution.

## Probe Types

| Type | Description |
|------|-------------|
| `tracepoint` | Captures call stack, local variables, async traces |
| `logpoint` | Evaluates expression (lightweight, no call stack); use `--log-expression` |
| `exceptionpoint` | Captures on exceptions |

**Unified API:** Use `debug list-probes`, `debug remove-probe`, `debug clear-probes`, `debug get-probe-snapshots`, and `debug clear-probe-snapshots` for tracepoints, logpoints, and (where applicable) watches/snapshots.

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

# With column (1-based)
browser-devtools-cli debug put-tracepoint --url-pattern "app.js" --line-number 42 --column-number 8
```

### List / Remove / Clear (unified)

```bash
# List all probes (tracepoints, logpoints, watches)
browser-devtools-cli debug list-probes

# List only tracepoints
browser-devtools-cli debug list-probes --types tracepoint

# Remove one probe by type and id (from list-probes)
browser-devtools-cli debug remove-probe --type tracepoint --id "tp_abc123"
browser-devtools-cli debug remove-probe --type logpoint --id "lp_xyz"
browser-devtools-cli debug remove-probe --type watch --id "w_abc"

# Clear probes (tracepoints, logpoints, watches). Omit types to clear all.
browser-devtools-cli debug clear-probes
browser-devtools-cli debug clear-probes --types tracepoint,logpoint
browser-devtools-cli debug clear-probes --types watches
```

### Get / Clear Snapshots (unified)

```bash
# Get all snapshots (tracepoint, logpoint, exceptionpoint)
browser-devtools-cli --json debug get-probe-snapshots

# Get only tracepoint snapshots
browser-devtools-cli --json debug get-probe-snapshots --types tracepoint

# Filter by probe id
browser-devtools-cli --json debug get-probe-snapshots --probe-id "tp_abc123"

# Polling: new snapshots since sequence 50
browser-devtools-cli --json debug get-probe-snapshots --from-sequence 50

# Limit results
browser-devtools-cli --json debug get-probe-snapshots --limit 10

# Clear snapshots (optional types and probeId)
browser-devtools-cli debug clear-probe-snapshots
browser-devtools-cli debug clear-probe-snapshots --types tracepoint,exceptionpoint
browser-devtools-cli debug clear-probe-snapshots --probe-id "tp_abc123"
```

**clear-probe-snapshots output (JSON):** `tracepointCleared`, `logpointCleared`, `exceptionpointCleared`, `message`.

## Logpoint Commands

### Put Logpoint

```bash
# Simple value
browser-devtools-cli debug put-logpoint \
  --url-pattern "app.js" \
  --line-number 42 \
  --log-expression "user.name"

# With column, condition, or hit condition
browser-devtools-cli debug put-logpoint \
  --url-pattern "app.js" --line-number 42 --log-expression "item" \
  --column-number 8 --condition "item.active" --hit-condition "> 5"

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

List/remove/clear: use `debug list-probes`, `debug remove-probe --type logpoint --id <id>`, `debug clear-probes`.

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

Get/clear: use `debug get-probe-snapshots --types exceptionpoint` and `debug clear-probe-snapshots --types exceptionpoint`.

## Watch Expressions

Watch expressions are evaluated at every tracepoint/exceptionpoint hit.

```bash
# Add watch expressions
browser-devtools-cli debug add-watch --expression "this"
browser-devtools-cli debug add-watch --expression "user.id"

# List watches (via list-probes)
browser-devtools-cli debug list-probes --types watch

# Remove one watch
browser-devtools-cli debug remove-probe --type watch --id "w_abc123"

# Clear all watches
browser-devtools-cli debug clear-probes --types watches
```

## Resolve Source Location

Resolve a generated/bundle code location to original source via source maps (e.g. minified stack to TypeScript).

```bash
browser-devtools-cli debug resolve-source-location \
  --url "http://localhost:3000/bundle.js" \
  --line 100

# With column (1-based)
browser-devtools-cli debug resolve-source-location --url "file:///dist/app.js" --line 42 --column 8
```

**Arguments:** `--url` (generated script URL), `--line` (1-based), `--column` (optional, 1-based). Returns `resolved`, `source`, `line`, `column`, `name` when a source map is available.

## Debug Status

```bash
browser-devtools-cli --json debug status
```

Returns: `enabled`, `hasSourceMaps`, `exceptionBreakpoint`, `tracepointCount`, `logpointCount`, `watchExpressionCount`, `snapshotStats`.

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
  "originalLocation": { "source": "src/app.ts", "line": 15, "column": 8 },
  "callStack": [...],
  "asyncStackTrace": { "segments": [...] },
  "watchResults": { "this": {...}, "arguments": [...] },
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

browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"
browser-devtools-cli $SESSION debug put-tracepoint \
  --url-pattern "app.js" \
  --line-number 100

# Trigger (e.g. click)
browser-devtools-cli $SESSION interaction click --selector "#submit-btn"

# Get snapshots
browser-devtools-cli $SESSION --json debug get-probe-snapshots --types tracepoint
```

### Catch Exceptions

```bash
browser-devtools-cli debug put-exceptionpoint --state uncaught
browser-devtools-cli navigation go-to --url "http://localhost:3000/buggy"
browser-devtools-cli --json debug get-probe-snapshots --types exceptionpoint
```

### Debug with Source Maps

```bash
browser-devtools-cli debug put-tracepoint \
  --url-pattern "src/components/Button.tsx" \
  --line-number 25
# Snapshots include originalLocation when source maps are available
```
