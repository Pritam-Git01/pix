# figma-mcp-chrome

A Claude Code skill for autonomous, pixel-perfect frontend development using Figma MCP and Chrome.

## What it does

The `/frontend` command launches an autopilot loop that:

1. Connects to Figma via MCP to extract design tokens, typography, and assets
2. Implements the UI in your codebase
3. Takes screenshots of both Figma and your local app
4. Compares pixel-by-pixel and auto-refines until they match

## Installation

Copy the `.claude/skills/frontend` folder into your project's `.claude/skills/` directory.

## Requirements

- [Figma MCP](https://github.com/anthropics/figma-mcp) configured in Claude Code
- Chrome with Claude extension for screenshots
- A frontend project with Tailwind CSS (recommended)

## Usage

```
/frontend
```

Then paste your Figma selection link when prompted.

## How it works

### Phase 0: System Verification
Checks MCP connection, Chrome extension, and clears your dev port.

### Phase 1: Context Gathering
Opens localhost and Figma, asks for the component link.

### Phase 2: Deep Execution
- Extracts tokens via `get_variable_defs`
- Syncs values to `tailwind.config.js`
- Maps icons to Lucide React
- Implements the component

### Phase 3: Recursive Refinement
Screenshots both Figma and local app, compares every pixel, and auto-fixes discrepancies until they match.

## License

MIT
