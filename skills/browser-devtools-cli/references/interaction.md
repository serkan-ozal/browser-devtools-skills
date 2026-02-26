# Interaction Tools

User interaction simulation commands. All tools accept **CSS selector or ref** (e.g. `e1`, `@e1`) from the last `a11y take-aria-snapshot`. Use refs when selectors are fragile or elements lack stable IDs.

## click

Click an element on the page.

```bash
browser-devtools-cli interaction click --selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to click |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Click by ref (from ARIA snapshot)
browser-devtools-cli a11y take-aria-snapshot
browser-devtools-cli interaction click --selector "e1"

# Click a button
browser-devtools-cli interaction click --selector "button#submit"

# Click by data-testid
browser-devtools-cli interaction click --selector "[data-testid='login-btn']"
```

---

## fill

Fill out an input field (clears existing content first).

```bash
browser-devtools-cli interaction fill --selector <selector-or-ref> --value <value>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for input field |
| `--value` | string | Yes | - | Value to fill |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Fill text input
browser-devtools-cli interaction fill --selector "#email" --value "user@example.com"

# Fill password
browser-devtools-cli interaction fill --selector "input[type=password]" --value "secret123"

# Fill textarea
browser-devtools-cli interaction fill --selector "textarea#message" --value "Hello world"
```

---

## hover

Hover over an element.

```bash
browser-devtools-cli interaction hover --selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to hover |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Hover over menu
browser-devtools-cli interaction hover --selector ".dropdown-menu"

# Hover to reveal tooltip
browser-devtools-cli interaction hover --selector "[data-tooltip]"
```

---

## select

Select an option from a dropdown.

```bash
browser-devtools-cli interaction select --selector <selector-or-ref> --value <value>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for select element |
| `--value` | string | Yes | - | Option value to select |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Select by value
browser-devtools-cli interaction select --selector "#country" --value "US"

# Select by visible text
browser-devtools-cli interaction select --selector "#size" --value "Large"
```

---

## press-key

Press a keyboard key. Supports optional hold and repeat (e.g. for scroll-by-key).

```bash
browser-devtools-cli interaction press-key --key <key> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--key` | string | Yes | - | Key to press (e.g., Enter, Tab, Escape, ArrowDown, Space) |
| `--selector` | string | No | - | CSS selector or ref (e1, @e1) to focus before pressing |
| `--hold-ms` | number | No | - | Ms between keydown and keyup |
| `--repeat` | boolean | No | `false` | Repeat key while holdMs (e.g. for scroll) |
| `--repeat-interval-ms` | number | No | `50` | Interval between repeated presses when repeat=true (min 10) |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for selector (if used) |

**Examples:**

```bash
# Press Enter
browser-devtools-cli interaction press-key --key "Enter"

# Press Tab to navigate
browser-devtools-cli interaction press-key --key "Tab"

# Press Escape to close modal
browser-devtools-cli interaction press-key --key "Escape"

# Press on specific element
browser-devtools-cli interaction press-key --selector "#search" --key "Enter"
```

---

## scroll

Scroll the page viewport or a scrollable element.

```bash
browser-devtools-cli interaction scroll [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--mode` | enum | No | `by` | `by` (dx/dy), `to` (x/y), `top`, `bottom`, `left`, `right` |
| `--selector` | string | No | - | CSS selector or ref for scroll container (omit = viewport) |
| `--dx` | number | No | - | Delta X for mode=by (pixels) |
| `--dy` | number | No | - | Delta Y for mode=by (pixels) |
| `--x` | number | No | - | Absolute X for mode=to |
| `--y` | number | No | - | Absolute Y for mode=to |
| `--behavior` | enum | No | `auto` | `auto` or `smooth` |

**Examples:**

```bash
# Scroll down by 500px
browser-devtools-cli interaction scroll --mode by --dy 500

# Scroll to bottom of page
browser-devtools-cli interaction scroll --mode bottom

# Scroll inside a container
browser-devtools-cli interaction scroll --selector "#sidebar" --mode bottom

# Scroll up
browser-devtools-cli interaction scroll --mode by --dy -200

# Smooth scroll to position
browser-devtools-cli interaction scroll --mode to --x 0 --y 1000 --behavior smooth
```

---

## drag

Drag an element to another location.

```bash
browser-devtools-cli interaction drag --source-selector <selector-or-ref> --target-selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--source-selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to drag |
| `--target-selector` | string | Yes | - | CSS selector or ref (e2, @e2) for drop target |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for elements |

**Examples:**

```bash
browser-devtools-cli interaction drag --source-selector ".draggable" --target-selector ".dropzone"

# Using refs from ARIA snapshot
browser-devtools-cli interaction drag --source-selector "e1" --target-selector "e2"
```

---

## resize-viewport

Resize the browser viewport.

```bash
browser-devtools-cli interaction resize-viewport --width <px> --height <px>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--width` | number | Yes | - | Viewport width in pixels |
| `--height` | number | Yes | - | Viewport height in pixels |

**Examples:**

```bash
# Mobile viewport
browser-devtools-cli interaction resize-viewport --width 375 --height 667

# Desktop viewport
browser-devtools-cli interaction resize-viewport --width 1920 --height 1080

# Tablet viewport
browser-devtools-cli interaction resize-viewport --width 768 --height 1024
```

---

## resize-window

Resize the real browser window (OS-level). Best with Chromium and headful mode.

```bash
browser-devtools-cli interaction resize-window [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--width` | number | No* | - | Window width in pixels (*required when state=normal) |
| `--height` | number | No* | - | Window height in pixels (*required when state=normal) |
| `--state` | enum | No | `normal` | `normal`, `maximized`, `minimized`, `fullscreen` (width/height ignored when not normal) |

## Form Automation Example

```bash
SESSION="--session-id form-test"

# Navigate to form
browser-devtools-cli $SESSION navigation go-to --url "https://example.com/signup"

# Fill registration form
browser-devtools-cli $SESSION interaction fill --selector "#firstName" --value "John"
browser-devtools-cli $SESSION interaction fill --selector "#lastName" --value "Doe"
browser-devtools-cli $SESSION interaction fill --selector "#email" --value "john@example.com"
browser-devtools-cli $SESSION interaction fill --selector "#password" --value "SecurePass123!"

# Select country
browser-devtools-cli $SESSION interaction select --selector "#country" --value "US"

# Check terms checkbox
browser-devtools-cli $SESSION interaction click --selector "#terms"

# Submit form
browser-devtools-cli $SESSION interaction click --selector "button[type=submit]"

# Wait for response
browser-devtools-cli $SESSION sync wait-for-network-idle

# Verify success
browser-devtools-cli $SESSION content get-as-text --selector ".success-message"
```

## Responsive Testing Example

```bash
SESSION="--session-id responsive-test"

# Navigate
browser-devtools-cli $SESSION navigation go-to --url "https://example.com"

# Test mobile
browser-devtools-cli $SESSION interaction resize-viewport --width 375 --height 667
browser-devtools-cli $SESSION content take-screenshot --name "mobile"

# Test tablet
browser-devtools-cli $SESSION interaction resize-viewport --width 768 --height 1024
browser-devtools-cli $SESSION content take-screenshot --name "tablet"

# Test desktop
browser-devtools-cli $SESSION interaction resize-viewport --width 1920 --height 1080
browser-devtools-cli $SESSION content take-screenshot --name "desktop"
```
