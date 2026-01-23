---
name: frontend
description: Launches an autonomous, pixel-perfect UI implementation loop using Figma MCP and Chrome.
allowed-tools: [Bash, mcp__figma__get_design_context, mcp__figma__get_image, mcp__figma__get_variable_defs, chrome_navigate, chrome_screenshot]
---

# /frontend: The Pixel-Perfect Autonomous Loop

## Phase 0: System Verification
1. **MCP Check**: Verify the Figma MCP is connected. If not, alert me.
2. **Chrome Check**: Ensure the Claude Chrome extension is active.
3. **Port Cleanup**: Identify the process on your dev port (e.g., 3000) using `lsof`. Kill it and restart the server (`npm run dev`) in the background.

## Phase 1: Context Gathering
1. Open Chrome tabs: `localhost:3000` and `figma.com`.
2. **Stop and Ask**: "Environment ready. Please paste the Figma 'Link to Selection' for the component we are building."

## Phase 2: The Magic Prompt (Deep Execution)
Once the link is provided, you must execute this EXACT sequence. Do not skip details:

### 1. Hard Data Extraction (Figma API)
* **Tokens**: Use `get_variable_defs`. Extract hex codes, corner-radius, and shadows.
* **Typography**: Get numeric `font-weight` (e.g., 600 vs 700), `line-height`, and `letter-spacing`.
* **Tailwind Sync**: Check `tailwind.config.js`. If a Figma value is missing, **update the config**. NEVER hardcode arbitrary hex values like `text-[#f3f3f3]` if they should be tokens.
* **Icons**: Map Figma layer names to **Lucide React**. Match the `stroke-width` and `size` (px) to the design exactly.

### 2. Implementation & "Building Brick" QA
Implement the code, then start the **Comparison Loop**:
1. **Screenshot App**: Take a capture of the rendered component in Chrome.
2. **Screenshot Figma**: Use `get_image` to get the high-res reference from Figma.
3. **The "Brick" Checklist**: Compare the following with 1:1 scrutiny:
    - **Titles**: Is the font boldness and vertical alignment identical?
    - **Icon Shapes**: Does the Lucide icon look exactly like the Figma version? Check the stroke thickness.
    - **Distances**: Measure the gaps/margins. If Figma says 24px and the app looks like 20px, refactor.
    - **Borders**: Verify 1px vs 2px lines and the exact curve of `border-radius`.

## Phase 3: Recursive Refinement
If you find ANY discrepancy (even 1px), you must:
- Explain what is wrong.
- Fix the code.
- **Repeat Phase 2, Step 3** (The Screenshot QA).

**Success Condition**: You are only finished when you can display a side-by-side screenshot proving the local app and Figma design are indistinguishable.

**ULTRA-THINK MODE ENABLED**: Take your time. I value perfection over speed.
