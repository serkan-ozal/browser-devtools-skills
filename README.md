# Browser DevTools Skills

Skills for [Browser DevTools MCP](https://github.com/serkan-ozal/browser-devtools-mcp) - AI agent skills for browser automation, testing, and debugging.

## Installation

Install browser automation capabilities as a skill for AI coding agents (Claude Code, Cursor, Windsurf, etc.) using the [skills.sh](https://skills.sh) ecosystem:

```bash
npx skills add serkan-ozal/browser-devtools-skills
```

## Available Skills

### CLI Skills

| Skill | Description |
|-------|-------------|
| [browser-devtools-cli](skills/browser-devtools-cli/SKILL.md) | Complete CLI reference for browser automation |
| [node-devtools-cli](skills/node-devtools-cli/SKILL.md) | CLI for Node.js backend debugging (tracepoints, logpoints, Inspector Protocol) |

**browser-devtools-cli Domain References:**
- [navigation](skills/browser-devtools-cli/references/navigation.md) - Page navigation (go-to, go-back-or-forward, reload)
- [content](skills/browser-devtools-cli/references/content.md) - Content extraction (screenshot, PDF, HTML, text)
- [interaction](skills/browser-devtools-cli/references/interaction.md) - User interactions (click, fill, hover, scroll)
- [a11y](skills/browser-devtools-cli/references/a11y.md) - Accessibility snapshots (ARIA, AX tree)
- [o11y](skills/browser-devtools-cli/references/o11y.md) - Observability (Web Vitals, console, HTTP, traces)
- [debug](skills/browser-devtools-cli/references/debug.md) - Non-blocking debugging (tracepoints, logpoints, exceptions)
- [run](skills/browser-devtools-cli/references/run.md) - JavaScript execution (browser, sandbox)
- [stub](skills/browser-devtools-cli/references/stub.md) - HTTP mocking (intercept, mock, clear)
- [sync](skills/browser-devtools-cli/references/sync.md) - Synchronization (wait for network idle)
- [react](skills/browser-devtools-cli/references/react.md) - React DevTools integration
- [figma](skills/browser-devtools-cli/references/figma.md) - Figma design comparison

**node-devtools-cli References:**
- [debug](skills/node-devtools-cli/references/debug.md) - Connection, tracepoints, logpoints, exceptionpoints, snapshots
- [run](skills/node-devtools-cli/references/run.md) - JavaScript execution in connected Node process

### Task-Specific Skills

| Skill | Description |
|-------|-------------|
| [browser-testing](skills/browser-testing/SKILL.md) | Browser automation, interaction, and form testing |
| [debugging](skills/debugging/SKILL.md) | Console, network, tracepoints, logpoints, and exception monitoring |
| [visual-testing](skills/visual-testing/SKILL.md) | Screenshots, responsive testing, and Figma comparison |
| [performance-audit](skills/performance-audit/SKILL.md) | Web Vitals and performance analysis |
| [accessibility-audit](skills/accessibility-audit/SKILL.md) | WCAG compliance and ARIA validation |
| [api-testing](skills/api-testing/SKILL.md) | API mocking and request interception |
| [react-debugging](skills/react-debugging/SKILL.md) | React component inspection |
| [observability](skills/observability/SKILL.md) | Distributed tracing and monitoring |

## Quick Start

```bash
# Install globally
npm install -g browser-devtools-mcp

# Browser automation
browser-devtools-cli navigation go-to --url "https://example.com"
browser-devtools-cli content take-screenshot --name "homepage"
browser-devtools-cli content get-as-text

# Node.js backend debugging (requires PLATFORM=node daemon; use --port 2021 if browser daemon is on 2020)
PLATFORM=node node-devtools-cli daemon start --port 2021
node-devtools-cli --port 2021 debug connect --pid 12345
node-devtools-cli --port 2021 debug put-tracepoint --url-pattern "server.js" --line-number 42
```

## License

Elastic License 2.0 (ELv2)
