# figma-mcp-chrome

Build frontend on autopilot. Pixel-perfect Figma-to-code with autonomous refinement.

## Install

```bash
claude plugin install skobak/figma-mcp-chrome
```

## Usage

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
- Recursive refinement until pixel-perfect

## Requirements

- [Figma MCP](https://www.npmjs.com/package/@anthropic/figma-mcp) configured
- Chrome with [Claude extension](https://chromewebstore.google.com/detail/claude)

## License

MIT
