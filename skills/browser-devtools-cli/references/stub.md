# Stub Tools

HTTP request interception and mocking commands.

## intercept-http-request

Intercept and modify outgoing HTTP requests.

```bash
browser-devtools-cli stub intercept-http-request --pattern <pattern> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--pattern` | string | Yes | - | URL glob pattern to match (picomatch) |
| `--modifications` | object | No | - | `headers`, `body`, `method` to apply to the request |
| `--delay-ms` | number | No | `0` | Delay before forwarding (ms) |
| `--times` | number | No | - | Number of times to apply (-1 = infinite) |

**Examples:**

```bash
# Add auth header to API calls
browser-devtools-cli stub intercept-http-request \
  --pattern "**/api/**" \
  --modifications '{"headers": {"Authorization": "Bearer token123"}}'

# Modify request body and method
browser-devtools-cli stub intercept-http-request \
  --pattern "**/api/submit" \
  --modifications '{"method": "POST", "body": {"modified": true}}'

# Add delay to simulate slow network
browser-devtools-cli stub intercept-http-request \
  --pattern "**/*" \
  --delay-ms 2000
```

---

## mock-http-response

Mock HTTP responses for matching requests.

```bash
browser-devtools-cli stub mock-http-response --pattern <pattern> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--pattern` | string | Yes | - | URL glob pattern to match (picomatch) |
| `--response` | object | No | - | `action` (fulfill|abort), `status`, `headers`, `body`, `abortErrorCode`. Or use shortcuts below. |
| `--status` | number | No | `200` | Response status (when action=fulfill) |
| `--headers` | object | No | - | Response headers |
| `--body` | string | No | - | Response body |
| `--delay-ms` | number | No | `0` | Delay before responding (ms) |
| `--times` | number | No | - | Number of times to apply (-1 = infinite) |
| `--chance` | number | No | `1` | Probability 0–1 (flaky testing). Omit = always. |

**Examples:**

```bash
# Mock API response
browser-devtools-cli stub mock-http-response \
  --pattern "**/api/users" \
  --body '[{"id": 1, "name": "Test User"}]' \
  --headers '{"Content-Type": "application/json"}'

# Simulate error response
browser-devtools-cli stub mock-http-response \
  --pattern "**/api/data" \
  --status 500 \
  --body '{"error": "Internal Server Error"}'

# Simulate network failure (abort request)
browser-devtools-cli stub mock-http-response \
  --pattern "**/api/**" \
  --response '{"action": "abort"}'

# Mock with delay (slow API)
browser-devtools-cli stub mock-http-response \
  --pattern "**/api/slow" \
  --delay-ms 3000 \
  --body '{"result": "delayed"}'

# Mock only first 3 requests
browser-devtools-cli stub mock-http-response \
  --pattern "**/api/limited" \
  --times 3 \
  --body '{"mocked": true}'
```

---

## list

List all active stubs.

```bash
browser-devtools-cli stub list
```

**Output (JSON):**

```json
{
  "stubs": [
    {
      "id": "stub-1",
      "kind": "intercept-http-request",
      "enabled": true,
      "pattern": "**/api/**",
      "delayMs": 0,
      "times": -1,
      "usedCount": 0
    },
    {
      "id": "stub-2",
      "kind": "mock-http-response",
      "enabled": true,
      "pattern": "**/api/users",
      "delayMs": 0,
      "times": -1,
      "usedCount": 3,
      "action": "fulfill",
      "status": 200
    }
  ]
}
```

---

## clear

Clear stubs.

```bash
browser-devtools-cli stub clear [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--stub-id` | string | No | - | Specific stub ID to clear (from `stub list`). Omit to clear all. |

**Examples:**

```bash
# Clear specific stub
browser-devtools-cli stub clear --stub-id "stub-1"

# Clear all stubs
browser-devtools-cli stub clear
```

**Output (JSON):** `clearedCount` — number of stubs removed.

## API Mocking Example

```bash
SESSION="--session-id mock-test"

# Setup mock before navigation
browser-devtools-cli $SESSION stub mock-http-response \
  --pattern "**/api/users" \
  --body '[{"id": 1, "name": "Test User"}]' \
  --headers '{"Content-Type": "application/json"}'

# Navigate (will use mocked data)
browser-devtools-cli $SESSION navigation go-to --url "https://app.example.com"

# Verify mock is active
browser-devtools-cli --json $SESSION stub list

# Test UI with mocked data
browser-devtools-cli $SESSION content take-screenshot --name "mocked-data"

# Cleanup
browser-devtools-cli $SESSION stub clear
```

## Error Handling Testing

```bash
SESSION="--session-id error-test"

# Navigate first
browser-devtools-cli $SESSION navigation go-to --url "https://app.example.com"

# Setup error mock
browser-devtools-cli $SESSION stub mock-http-response \
  --pattern "**/api/submit" \
  --status 500 \
  --body '{"error": "Server Error"}'

# Trigger form submission
browser-devtools-cli $SESSION interaction click --selector "#submit-btn"

# Wait for error handling
browser-devtools-cli $SESSION sync wait-for-network-idle

# Verify error UI
browser-devtools-cli $SESSION content get-as-text --selector ".error-message"
browser-devtools-cli $SESSION content take-screenshot --name "error-state"

# Cleanup
browser-devtools-cli $SESSION stub clear
```

## Offline Testing

```bash
SESSION="--session-id offline-test"

# Navigate while online
browser-devtools-cli $SESSION navigation go-to --url "https://app.example.com"

# Simulate offline by aborting all requests
browser-devtools-cli $SESSION stub mock-http-response \
  --pattern "**/*" \
  --response '{"action": "abort"}'

# Test offline behavior
browser-devtools-cli $SESSION interaction click --selector "#refresh-data"
browser-devtools-cli $SESSION content take-screenshot --name "offline-state"

# Cleanup
browser-devtools-cli $SESSION stub clear
```

## Slow Network Testing

```bash
SESSION="--session-id slow-test"

# Add 3 second delay to all requests
browser-devtools-cli $SESSION stub intercept-http-request \
  --pattern "**/*" \
  --delay-ms 3000

# Navigate and observe loading states
browser-devtools-cli $SESSION navigation go-to --url "https://app.example.com"

# Screenshot during loading
browser-devtools-cli $SESSION content take-screenshot --name "loading-state"

# Wait for completion
browser-devtools-cli $SESSION sync wait-for-network-idle --timeout-ms 60000

# Screenshot after load
browser-devtools-cli $SESSION content take-screenshot --name "loaded-state"

# Cleanup
browser-devtools-cli $SESSION stub clear
```
