# React Tools

React DevTools integration commands.

## Prerequisites

- **Persistent browser context required:** Start with `--persistent` flag
- **React DevTools extension:** Must be manually installed in the browser profile
  - Chrome Web Store: [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- Without the extension, tools fall back to best-effort DOM scanning

## get-component-for-element

Find React component(s) associated with a DOM element.

```bash
browser-devtools-cli react get-component-for-element --selector <selector>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | No | - | CSS selector for DOM element (or use --x/--y for viewport coords) |
| `--x` | number | No | - | Viewport X when selector omitted |
| `--y` | number | No | - | Viewport Y when selector omitted |
| `--max-stack-depth` | number | No | 30 | Max component stack depth |
| `--include-props-preview` | boolean | No | true | Include props preview in output |
| `--max-props-preview-chars` | number | No | 2000 | Max chars for props preview |

**Description:**

Uses React Fiber to find the component tree for a given DOM element. Returns:
- Component name
- Props
- State (if available)
- Source file location (in development builds)

**Examples:**

```bash
# Find component for a button
browser-devtools-cli react get-component-for-element --selector "button.submit"

# Find component for a specific element (JSON for parsing)
browser-devtools-cli --json react get-component-for-element --selector "[data-testid='user-card']"

# Find component for form
browser-devtools-cli --json react get-component-for-element --selector "form#login"
```

**Output (JSON):**

```json
{
  "element": "button.submit",
  "components": [
    {
      "name": "SubmitButton",
      "props": { "disabled": false, "loading": false },
      "source": "src/components/SubmitButton.tsx:15"
    },
    {
      "name": "Form",
      "props": { "onSubmit": "[Function]" }
    }
  ]
}
```

---

## get-element-for-component

Find DOM elements rendered by a React component.

```bash
browser-devtools-cli react get-element-for-component --component-name <name>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--component-name` | string | No | - | Component name to search for (use with anchor for best match) |
| `--anchor-selector` | string | No | - | Anchor element selector/ref to target instance |
| `--anchor-x` | number | No | - | Viewport X when anchor-selector omitted |
| `--anchor-y` | number | No | - | Viewport Y when anchor-selector omitted |
| `--match-strategy` | enum | No | `exact` | `exact` or `contains` for component name |
| `--file-name-hint` | string | No | - | Source file hint for disambiguation |
| `--line-number` | number | No | - | Line hint for disambiguation |
| `--max-elements` | number | No | 25 | Max DOM elements to return |
| `--only-visible` | boolean | No | true | Only include visible elements |
| `--only-in-viewport` | boolean | No | false | Only include elements in viewport |
| `--max-matches` | number | No | 10 | Max component instances to consider |

**Description:**

Searches the React component tree and returns DOM elements rendered by matching components. Prefer anchor (selector or x,y) plus component name for reliable instance targeting.

**Examples:**

```bash
# Find elements rendered by Button component
browser-devtools-cli react get-element-for-component --component-name "Button"

# Find elements by component name (JSON for parsing)
browser-devtools-cli --json react get-element-for-component --component-name "UserCard"

# Find specific component instance
browser-devtools-cli --json react get-element-for-component --component-name "Header"
```

**Output (JSON):**

```json
{
  "component": "Button",
  "elements": [
    {
      "selector": "button.primary-btn",
      "tagName": "button",
      "textContent": "Submit",
      "boundingBox": { "x": 100, "y": 200, "width": 120, "height": 40 }
    }
  ]
}
```

## Setup Instructions

### 1. First Run Setup (with visible browser)

```bash
# Start with persistent context and visible browser
browser-devtools-cli --no-headless --persistent navigation go-to --url "chrome://extensions"
```

Then manually install React DevTools from Chrome Web Store.

### 2. Subsequent Runs

```bash
# Persistent context preserves the extension
SESSION="--session-id react-debug --persistent --json"

# Navigate to React app
browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"
```

## React Debugging Example

```bash
SESSION="--session-id react-debug --persistent --json"

# Navigate to React app
browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"
browser-devtools-cli $SESSION sync wait-for-network-idle

# Find component for specific element
browser-devtools-cli $SESSION react get-component-for-element --selector "[data-testid='user-profile']"

# Find where a component renders
browser-devtools-cli $SESSION react get-element-for-component --component-name "ErrorBoundary"

# Screenshot component
browser-devtools-cli $SESSION content take-screenshot --selector "[data-testid='user-profile']" --name "user-profile"
```

## Component Props Inspection

```bash
SESSION="--session-id props-test --persistent --json"

browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"

# Get component info with props
COMPONENT=$(browser-devtools-cli $SESSION react get-component-for-element --selector ".dashboard")
echo "Component: $COMPONENT"

# Extract specific prop value using jq
echo $COMPONENT | jq '.components[0].props'
```

## Find All Instances of Component

```bash
SESSION="--session-id find-all --persistent --json"

browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"

# Find all Button instances
BUTTONS=$(browser-devtools-cli $SESSION react get-element-for-component --component-name "Button")
echo "Found buttons: $BUTTONS"

# Count instances
echo $BUTTONS | jq '.elements | length'
```

## Combined with Accessibility

```bash
SESSION="--session-id react-a11y --persistent --json"

browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000"

# Get React component info
browser-devtools-cli $SESSION react get-component-for-element --selector "nav"

# Check accessibility of the same element
browser-devtools-cli $SESSION a11y take-aria-snapshot --selector "nav"
```

## Limitations

- Requires React DevTools extension for full functionality
- Component names may be minified in production builds
- State access depends on React version and build configuration
- Source locations only available in development builds
- Falls back to DOM scanning without extension (less reliable)
