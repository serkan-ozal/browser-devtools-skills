# Node.js Debug Commands

Non-blocking debugging for Node.js backend processes. Connect first, then set probes.

**Unified API:** Use `debug list-probes`, `debug remove-probe`, `debug clear-probes`, `debug get-probe-snapshots`, and `debug clear-probe-snapshots` for tracepoints, logpoints, and (where applicable) watches/snapshots.

## Connection

### Connect

MCP parameters: `pid`, `processName`, `containerId`, `containerName`, `host`, `inspectorPort`, `wsUrl`.

```bash
# By PID
node-devtools-cli debug connect --pid 12345

# By process name
node-devtools-cli debug connect --process-name "server.js"

# By Docker container
node-devtools-cli debug connect --container-name my-app --inspector-port 9229
node-devtools-cli debug connect --container-id <id> --host host.docker.internal --inspector-port 9229

# By inspector port (already --inspect)
node-devtools-cli debug connect --inspector-port 9229

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

# List / Remove / Clear (unified)
node-devtools-cli debug list-probes --types tracepoint
node-devtools-cli debug remove-probe --type tracepoint --id "tp_abc123"
node-devtools-cli debug clear-probes --types tracepoint
```

## Logpoints

Optional: `--column-number`, `--condition`, `--hit-condition` (same semantics as tracepoint).

```bash
node-devtools-cli debug put-logpoint \
  --url-pattern "utils.ts" \
  --line-number 10 \
  --log-expression "`Processing: ${item}`"

# With condition / hit condition
node-devtools-cli debug put-logpoint \
  --url-pattern "utils.ts" --line-number 10 --log-expression "item" \
  --condition "item.active" --hit-condition "> 5"

# List / Remove / Clear (unified)
node-devtools-cli debug list-probes --types logpoint
node-devtools-cli debug remove-probe --type logpoint --id "lp_xyz"
node-devtools-cli debug clear-probes --types logpoint
```

## Exceptionpoints

```bash
node-devtools-cli debug put-exceptionpoint --state uncaught
node-devtools-cli debug put-exceptionpoint --state all
node-devtools-cli debug put-exceptionpoint --state none
```

## Snapshots (unified)

```bash
# Get all or by type
node-devtools-cli --json debug get-probe-snapshots
node-devtools-cli --json debug get-probe-snapshots --types tracepoint,logpoint,exceptionpoint
node-devtools-cli --json debug get-probe-snapshots --probe-id "tp_123"
node-devtools-cli --json debug get-probe-snapshots --from-sequence 50 --limit 20

# Clear snapshots
node-devtools-cli debug clear-probe-snapshots
node-devtools-cli debug clear-probe-snapshots --types tracepoint --probe-id "tp_123"
```

## Watch Expressions

```bash
node-devtools-cli debug add-watch --expression "req.body"
node-devtools-cli debug add-watch --expression "this.state"

# List / Remove / Clear (unified)
node-devtools-cli debug list-probes --types watch
node-devtools-cli debug remove-probe --type watch --id "w_abc"
node-devtools-cli debug clear-probes --types watches
```

## Resolve Source Location

For source map resolution (generated → original source). Input: generated script URL, line, optional column (1-based).

```bash
node-devtools-cli debug resolve-source-location \
  --url "file:///path/to/dist/app.js" \
  --line 100

# With column
node-devtools-cli debug resolve-source-location --url "file:///dist/app.js" --line 100 --column 5
```

## Console Logs (get-logs)

Same filtering as browser `o11y get-console-messages`: single `type` (level or higher), `search`, optional `timestamp`, `sequenceNumber`, `limit` (count + from).

```bash
node-devtools-cli --json debug get-logs
node-devtools-cli --json debug get-logs --search "error"
node-devtools-cli --json debug get-logs --type error
node-devtools-cli --json debug get-logs --type warning
```

**Optional:** `--timestamp`, `--sequence-number`, `--limit` (JSON object: `{"count":100,"from":"end"}`; count 0 = no limit). `--type` values: `all`, `debug`, `info`, `warning`, `error` (filter by this level or higher).
