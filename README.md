# figma-mcp-chrome

Build frontend on autopilot. Pixel-perfect Figma-to-code with autonomous refinement.

## Install

```bash
claude plugin install github:skobak/figma-mcp-chrome
```

Or manually:
```bash
git clone https://github.com/skobak/figma-mcp-chrome.git ~/.claude/plugins/figma-mcp-chrome
```

## Usage

Start Claude Code with Chrome integration:

```bash
claude --dangerously-skip-permissions --chrome
```

Then run:

```
/frontend
```

## How it works

1. **Run `/frontend`** — plugin checks your Figma MCP and Chrome extension setup, guides you if anything is missing

2. **Auto-launch** — starts your dev server, opens localhost and Figma in Chrome, asks you to navigate to the component and paste the Figma link

3. **Sit back** — watch it extract tokens, implement the component, screenshot both views, compare pixel-by-pixel, and auto-fix until perfect

## What it handles

- Detects your stack (Vite, Next, etc.)
- Finds your design system (Tailwind, styled-components, Chakra, etc.)
- Identifies your icon library
- Syncs Figma tokens to your config
- Uses Code Connect to reuse existing components
- Recursive refinement until pixel-perfect

## Figma MCP Tools Used

| Tool | Purpose |
|------|---------|
| `whoami` | Verify authentication |
| `get_metadata` | Component structure |
| `get_design_context` | Full design spec |
| `get_variable_defs` | Design tokens |
| `get_code_connect_map` | Existing component mappings |
| `get_screenshot` | Figma reference image |

## Requirements

- **Claude Code** v2.0.73+ with `--chrome` flag
- **Claude in Chrome** extension v1.0.36+ ([install](https://claude.com/chrome))
- **Figma MCP** configured ([setup guide](https://help.figma.com/hc/en-us/articles/32132100833559))
- **Paid Claude plan** (Pro, Team, or Enterprise)

## License

MIT
