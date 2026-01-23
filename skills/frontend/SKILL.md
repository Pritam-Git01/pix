---
name: frontend
description: Launches an autonomous, pixel-perfect UI implementation loop using Figma MCP and Chrome.
allowed-tools: [Bash, Read, Glob, Grep, mcp__figma__whoami, mcp__figma__get_design_context, mcp__figma__get_image, mcp__figma__get_variable_defs, chrome_navigate, chrome_screenshot]
---

# /frontend: The Pixel-Perfect Autonomous Loop

## Phase 0: Project Discovery

Before anything else, analyze the project to understand its stack:

### 1. Package Manager Detection
Check which lockfile exists: `package-lock.json` (npm), `yarn.lock` (yarn), `pnpm-lock.yaml` (pnpm), `bun.lockb` (bun).
Use the corresponding command for all package operations.

### 2. Dev Server & Port Detection
- Read `package.json` scripts to find the dev command
- Check for port configuration in: `vite.config.*`, `next.config.*`, `nuxt.config.*`, `webpack.config.*`, `.env*`, or the dev script itself
- Common patterns: `--port`, `-p`, `PORT=`, `server.port`
- Default fallback order: 5173 (Vite), 3000 (Next/CRA), 8080 (Vue CLI)

### 3. Design System Detection
Scan `package.json` dependencies for styling approach:
- **Tailwind**: `tailwindcss` → config in `tailwind.config.*`
- **CSS-in-JS**: `styled-components`, `@emotion/*`, `stitches`
- **Component Libraries**: `@chakra-ui/*`, `@mui/*`, `@mantine/*`, `antd`, `@radix-ui/*`
- **CSS Modules**: check for `*.module.css` files
- **Vanilla CSS/SCSS**: check for global stylesheets

### 4. Icon Library Detection
Scan `package.json` and imports for icon libraries:
- `lucide-react` → Lucide
- `@heroicons/react` → Heroicons
- `react-icons` → React Icons (multi-library)
- `@radix-ui/react-icons` → Radix Icons
- `@fortawesome/*` → FontAwesome
- `@phosphor-icons/react` → Phosphor
- `@tabler/icons-react` → Tabler Icons
- If none found, ask user which to install or use inline SVGs

### 5. System Verification
1. **MCP Check**: Verify Figma MCP is connected and authenticated:
   - Call `mcp__figma__whoami` to check authentication status
   - If not connected: alert user to configure Figma MCP
   - If not authenticated: guide user to authenticate via Figma OAuth
   - Display the authenticated user info to confirm correct account
2. **Chrome Check**: Ensure Claude Chrome extension is active.
3. **Port Cleanup**: Using detected port, run `lsof -i :<PORT>`. Kill existing process if needed. Start dev server in background using detected package manager and script.

## Phase 1: Context Gathering

1. Start dev server and open Chrome with `localhost:<DETECTED_PORT>`
2. Open Figma in Chrome tab
3. **Stop and Ask**:
   - "Please log in to Figma if you haven't already"
   - "Navigate to the component you want to build"
   - "Right-click the component → 'Copy link to selection'"
   - "Paste the Figma link here"

## Phase 2: The Magic Prompt (Deep Execution)

Once the link is provided, you must execute this EXACT sequence. Do not skip details:

### 1. Hard Data Extraction (Figma API)
* **Tokens**: Use `get_variable_defs`. Extract hex codes, corner-radius, and shadows.
* **Typography**: Get numeric `font-weight` (e.g., 600 vs 700), `line-height`, and `letter-spacing`.
* **Design System Sync**:
  - If **Tailwind**: Check `tailwind.config.*`. If a Figma value is missing, **update the config**. NEVER hardcode arbitrary hex values like `text-[#f3f3f3]` if they should be tokens.
  - If **CSS-in-JS**: Add tokens to the theme object/file. NEVER use inline hex values.
  - If **Component Library**: Map to existing theme tokens, extend theme if needed. NEVER bypass the theme.
  - If **CSS/SCSS**: Add CSS custom properties to `:root`. NEVER scatter magic values.
* **Icons**: Map Figma layer names to the **project's detected icon library**. Match the `stroke-width` and `size` (px) to the design exactly. If no icon library exists, ask user preference or use inline SVG from Figma.

### 2. Implementation & "Building Brick" QA
Implement the code using the project's existing patterns, then start the **Comparison Loop**:
1. **Screenshot App**: Take a capture of the rendered component in Chrome.
2. **Screenshot Figma**: Use `get_image` to get the high-res reference from Figma.
3. **The "Brick" Checklist**: Compare the following with 1:1 scrutiny:
    - **Titles**: Is the font boldness and vertical alignment identical?
    - **Icon Shapes**: Does the icon look exactly like the Figma version? Check the stroke thickness.
    - **Distances**: Measure the gaps/margins. If Figma says 24px and the app looks like 20px, refactor.
    - **Borders**: Verify 1px vs 2px lines and the exact curve of `border-radius`.

## Phase 3: Recursive Refinement

If you find ANY discrepancy (even 1px):
1. Explain what is wrong.
2. Fix the code.
3. **Repeat Phase 2, Step 3** (Screenshot QA).

**Success Condition**: Only finished when side-by-side screenshots prove local app and Figma design are indistinguishable.

**ULTRA-THINK MODE ENABLED**: Take your time. Perfection over speed.
