# figma-mcp-chrome

Build frontend on autopilot. Pixel-perfect Figma-to-code with autonomous refinement.

## Install

```bash
git clone https://github.com/skobak/figma-mcp-chrome.git ~/.claude/skills/figma-mcp-chrome
```

Or copy `.claude/skills/frontend` into your project.

## Usage

```
/frontend
```

That's it. The skill handles everything:

- Verifies Figma MCP connection
- Checks Chrome extension is active
- Detects your dev server port
- Finds your design system (Tailwind, styled-components, Chakra, etc.)
- Identifies your icon library
- Starts your dev server
- Extracts tokens from Figma
- Implements the component
- Screenshots both Figma and localhost
- Compares pixel-by-pixel
- Auto-fixes until they match

## Requirements

- [Figma MCP](https://www.npmjs.com/package/@anthropic/figma-mcp) configured
- Chrome with [Claude extension](https://chromewebstore.google.com/detail/claude)

## License

MIT
