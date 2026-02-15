# Node.js Debug Commands

Non-blocking debugging for Node.js backend processes. Connect first, then set probes.

## Connection

### Connect

```bash
# By PID
node-devtools-cli debug connect --pid 12345

# By process name
node-devtools-cli debug connect --process-name "server.js"

# By Docker container
node-devtools-cli debug connect --container-name my-app --port 9229

# By inspector port (already --inspect)
node-devtools-cli debug connect --port 9229

# Direct WebSocket URL
node-devtools-cli debug connect --ws-url "ws://127.0.0.1:9229/abc-123"
```

### Disconnect

```bash
node-devtools-cli debug disconnect
```

### Status

```bash
node-devtools-cli --json debug status
```

Returns: `connected`, `enabled`, `pid`, `hasSourceMaps`, `exceptionBreakpoint`, `tracepointCount`, `logpointCount`, `watchExpressionCount`, `snapshotStats`.

## Tracepoints

`urlPattern` matches **script file paths** (e.g., `server.js`, `routes/api.ts`). Auto-escaped.

```bash
# Put tracepoint
node-devtools-cli debug put-tracepoint \
  --url-pattern "server.js" \
  --line-number 42

# With condition
node-devtools-cli debug put-tracepoint \
  --url-pattern "routes/users.ts" \
  --line-number 15 \
  --condition "req.user.id === 1"

# List / Remove / Clear
node-devtools-cli debug list-tracepoints
node-devtools-cli debug remove-tracepoint --id "tp_abc123"
node-devtools-cli debug clear-tracepoints
```

## Logpoints

```bash
node-devtools-cli debug put-logpoint \
  --url-pattern "utils.ts" \
  --line-number 10 \
  --log-expression "`Processing: ${item}`"

node-devtools-cli debug list-logpoints
node-devtools-cli debug remove-logpoint --id "lp_xyz"
node-devtools-cli debug clear-logpoints
```

## Exceptionpoints

```bash
node-devtools-cli debug put-exceptionpoint --state uncaught
node-devtools-cli debug put-exceptionpoint --state all
node-devtools-cli debug put-exceptionpoint --state none
```

## Snapshots

```bash
# Tracepoint
node-devtools-cli --json debug get-tracepoint-snapshots
node-devtools-cli --json debug get-tracepoint-snapshots --probe-id "tp_123"
node-devtools-cli --json debug get-tracepoint-snapshots --from-sequence 50
node-devtools-cli debug clear-tracepoint-snapshots

# Logpoint
node-devtools-cli --json debug get-logpoint-snapshots
node-devtools-cli debug clear-logpoint-snapshots

# Exceptionpoint
node-devtools-cli --json debug get-exceptionpoint-snapshots
node-devtools-cli debug clear-exceptionpoint-snapshots
```

## Watch Expressions

```bash
node-devtools-cli debug add-watch --expression "req.body"
node-devtools-cli debug add-watch --expression "this.state"
node-devtools-cli debug list-watches
node-devtools-cli debug remove-watch --id "w_abc"
node-devtools-cli debug clear-watches
```

## Resolve Source Location

For source map resolution (generated → original source):

```bash
node-devtools-cli debug resolve-source-location \
  --url "file:///path/to/dist/app.js" \
  --line 100
```

## Console Logs

```bash
node-devtools-cli --json debug get-logs
node-devtools-cli --json debug get-logs --search "error"
node-devtools-cli --json debug get-logs --type error
```
