---
name: pix
description: Launches an autonomous, pixel-perfect UI implementation loop using Figma MCP and Chrome.
allowed-tools: [Bash, Read, Glob, Grep, Edit, Write]
---

# /pix: The Pixel-Perfect Autonomous Loop

> **Note**: This skill requires Figma MCP and Claude Chrome extension. Tool names may vary based on your MCP configuration (e.g., `figma__get_screenshot` or `mcp__figma__get_screenshot`). Run `/mcp` to see available tools.

## Resource Strategy

### Tool Costs

Figma MCP tools have vastly different costs:

| Tool | Cost | Use For |
|------|------|---------|
| `get_metadata` | **Cheap** | Node tree with child IDs, positions, sizes. No styling. |
| `get_variable_defs` | **Cheap** | All design tokens (colors, spacing, radii) as name→value. |
| `get_code_connect_map` | **Cheap** | Check which Figma nodes map to existing code components. |
| `get_screenshot` | **Medium** | Visual image of a specific node. Target a `nodeId` to effectively crop/zoom. |
| `get_design_context` | **Expensive** | Full code + styling + assets. NEVER call on a large parent — call on individual sections/components. |

**Core rule**: Work top-down. Cheap calls first to build a map, then expensive calls only on the smallest necessary nodes.

### Image Budget

Claude API limits images to 2000px per dimension when >20 images are in a conversation. The limit is per-conversation — old images cannot be individually removed.

**Rules:**
- **~20 screenshots per conversation** before the 2000px resolution limit kicks in
- **Maintain a 3-image working set**: Figma layout (overview), Figma detail (current section), Chrome state (rendered result)
- **Never re-screenshot a static Figma node** — Figma designs don't change mid-session
- **Use `getComputedStyle` for all numerical properties** — zero image cost for verifying spacing, colors, fonts, sizes
- **Recovery**: `/compact` drops all old images from context (see Recovery section below)

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
   - Call `whoami` to check authentication status
   - If not connected: alert user to configure Figma MCP
   - If not authenticated: guide user to authenticate via Figma OAuth
   - Display the authenticated user info to confirm correct account
2. **Chrome Check**: Ensure Claude Chrome extension is active and connected.
3. **Dev Server**: Using detected port, run `lsof -i :<PORT>` to check if server is already running. If running, leave it alone. If not running, start dev server in background using detected package manager and script.

## Phase 1: Context Gathering

### Chrome Setup

Open Chrome with `localhost:<DETECTED_PORT>`:
- Navigate to the page/route where the component will live
- This tab is used for visual comparison and interaction testing (hover, click, focus states)

### User Input

**Stop and Ask**:
- "Paste the Figma link to the component you want to build"
- (Use 'Copy link to selection' in Figma)

> **Note**: No need to open Figma in Chrome — Figma MCP handles screenshots via `get_screenshot`.

## Phase 2: Reconnaissance (Cheap Calls Only)

Once the Figma link is provided, extract `fileKey` and `nodeId` from the URL. Then run ONLY cheap calls to build a complete map before writing any code.

### Step 1: Get the Node Tree

Call `get_metadata(nodeId, fileKey)`. This returns an XML tree like:

```xml
<Frame id="45:1" name="Header" type="FRAME" x="0" y="0" width="1440" height="80">
  <Component id="45:10" name="Logo" x="20" y="16" width="120" height="48"/>
  <Frame id="45:15" name="NavLinks" x="400" y="20" width="600" height="40">
    <Text id="45:16" name="Home" x="0" y="0" width="60" height="40"/>
  </Frame>
</Frame>
```

From this, build a **mental map**:
- Identify every major section (header, body, footer, sidebar, cards, etc.)
- Note each section's `nodeId` — you will use these to drill down
- Note positions and sizes to understand the overall layout grid
- Identify which children are components vs frames vs text vs icons

### Step 2: Extract All Design Tokens (Once)

Call `get_variable_defs(nodeId, fileKey)`. This returns ALL tokens:
```json
{"brand/primary": "#ff611c", "spacing/md": "16px", "radius/lg": "12px"}
```

Save these mentally. You will reference them throughout — do NOT call this again.

### Step 3: Check Existing Components

Call `get_code_connect_map(nodeId, fileKey)`. This tells you which Figma nodes already have code components in the codebase:
```json
{"45:10": {"codeConnectSrc": "src/components/Logo.tsx", "codeConnectName": "Logo"}}
```

Any matched component should be **reused**, not reimplemented.

### Step 4: Visual Overview (Image 1 of your budget)

Call `get_screenshot(nodeId, fileKey)` on the root selection. This gives you the full visual reference for understanding layout, proportions, and overall structure.

**This is your layout reference — do NOT retake it.** Figma designs are static within a session. Targeting child nodes via `nodeId` gives you an effective crop/zoom without any extra parameters.

**At this point you have NOT called `get_design_context` at all.** You have a complete structural map, all tokens, reusable component info, and a visual reference — all from cheap calls.

## Phase 3: Study & Implement (Code in the Dark)

Work like an efficient frontend developer: study the design deeply, memorize every detail, then code from memory without looking back.

### Step 1: Layout Shell

Using the metadata tree (positions, sizes, hierarchy), implement the outer layout structure:
- Container dimensions and positioning (flex, grid, etc.)
- Major section placement
- Background colors (from tokens extracted in Phase 2)
- Borders and dividers between sections

For the layout, the metadata positions + sizes + your visual overview screenshot are usually sufficient. Only call `get_design_context` if you need specific flex/grid properties you can't infer from the structure.

### Step 2: Study Every Section

For each major section identified in the metadata tree:

1. **Figma Detail**: Call `get_screenshot(sectionNodeId, fileKey)` — this crops to just that section, giving you a zoomed-in visual reference.

2. **Design Context**: Call `get_design_context(sectionNodeId, fileKey)` — this returns code and styling for ONLY this section (text only, no image cost).

3. **Absorb every detail**: fonts, sizes, weights, colors, spacing, borders, shadows, icon shapes. Burn it into memory. You will NOT look at Figma again until the review phase.

4. **Color sanity check**: Compare the colors from `get_design_context` against what you see in the Figma screenshot. If the design context shows raw color values but the screenshot reveals opacity layering, overlapping fills, or gradients — compute the final perceived color (e.g., a white text at 60% opacity on a dark background is NOT `#ffffff`). Use the visual truth, not the raw token.

**Do NOT take any Chrome screenshots during this phase.** You are studying, not checking.

**Key rule**: NEVER call `get_design_context` on the root selection. Always call it on the smallest meaningful node — a section, a card, a button group. This keeps each response small and focused.

### Step 3: Code from Memory

Now implement everything — all sections, all elements — using what you memorized:
- The design context output for exact properties
- Tokens from Phase 2 Step 2 (do not re-extract)
- Reusable components from Phase 2 Step 3

**Zero screenshots during coding.** You studied the design. You know every pixel. Code like you're working in the dark — every detail already committed to memory.

**Cheat move**: If you genuinely can't recall a specific detail (e.g., exact icon shape, a nested layout you didn't drill into, a subtle gradient), take ONE targeted `get_screenshot` on that specific Figma node. Then go back to coding. Don't use this as an excuse to screenshot everything — it's a surgical peek, not a second study phase.

### Step 4: Design System Sync

NEVER hardcode values. Always sync to the project's design system:

- If **Tailwind**: Check `tailwind.config.*`. If a Figma token is missing, **update the config**. NEVER hardcode arbitrary hex values like `text-[#f3f3f3]` if they should be tokens.
- If **CSS-in-JS**: Add tokens to the theme object/file. NEVER use inline hex values.
- If **Component Library**: Map to existing theme tokens, extend theme if needed. NEVER bypass the theme.
- If **CSS/SCSS**: Add CSS custom properties to `:root`. NEVER scatter magic values.

### Step 5: Icons

Map Figma layer names to the **project's detected icon library**. Match the `stroke-width` and `size` (px) to the design exactly. If no icon library exists, ask user preference or use inline SVG from Figma.

**Fallback**: If the Figma layer name doesn't map to an obvious icon library match, call `get_screenshot(iconNodeId, fileKey)` on that specific icon node and identify it visually. Match by shape, not by name.

### Property Checklist

Before writing code for ANY element, verify ALL applicable properties from the design context:

**Text**: `font-family`, `font-size`, `font-weight`, `line-height`, `letter-spacing`, `color`, `opacity`, `text-align`, `text-decoration`, `text-transform`

**Container/Block**: `width`, `height`, `min-width`, `max-width`, `padding` (all 4 sides), `margin`, `background-color`, `border-radius`, `border-width`, `border-color`, `border-style`, `box-shadow`, `opacity`, `overflow`

**Icon**: `size` (width/height), `fill`, `stroke`, `stroke-width`, `color` (independent from parent text)

**Button/Link**: All text + container properties + `cursor`, `hover-state`, `active-state`, `disabled-state`

**Image**: `width`, `height`, `object-fit`, `border-radius`, `border`, `aspect-ratio`

**Spacing**: `gap`, `row-gap`, `column-gap`, space-between elements

**Layout Principle**: Avoid hardcoded sizes (`max-w-[140px]`, `w-[200px]`, etc.). With correct `font-size`, `line-height`, `padding`, and parent container width, elements should naturally render correctly. Hardcoded dimensions are a symptom of incorrect upstream layout — fix the root cause instead.

**Responsive Design**: Keep responsiveness in mind. Even if only one breakpoint is provided in Figma, consider how the component should adapt to different screen sizes. When possible, implement both desktop and mobile-friendly styles using the project's responsive approach (Tailwind breakpoints, CSS media queries, container queries).

**Never use approximate Tailwind classes** (like `text-zinc-500`) when exact hex values are available from tokens.

## Phase 4: Refinement Loop (Until Pixel-Perfect)

You think you're done. Now prove it. This is an iterative loop — keep cycling until the result is perfect.

### The Loop

**Step 1: Take a Chrome screenshot (1 image)**

This is your first look at what you built. Compare it against the Figma overview and detail screenshots already in context. Be extremely picky. Look for:
- Visual alignment issues
- Missing elements or wrong proportions
- Layout shifts, spacing that looks off
- Wrong icon shapes
- Color or weight mismatches
- **Color comparison**: Sample the dominant colors from the Figma screenshot vs the Chrome screenshot. Background tones, text colors, border shades — if anything looks off, flag it even if the hex values technically match (opacity/layering can cause perceived differences)

**Step 2: Run numerical audit (zero images)**

For every element, run `getComputedStyle` in Chrome and compare against Figma design context / token values:

```js
const el = document.querySelector('.target-element');
const s = getComputedStyle(el);
JSON.stringify({
  padding: s.padding,
  margin: s.margin,
  gap: s.gap,
  backgroundColor: s.backgroundColor,
  color: s.color,
  fontSize: s.fontSize,
  fontWeight: s.fontWeight,
  lineHeight: s.lineHeight,
  letterSpacing: s.letterSpacing,
  borderRadius: s.borderRadius,
  borderWidth: s.borderWidth,
  borderColor: s.borderColor,
  boxShadow: s.boxShadow
});
```

Compare each value against the Figma token or design context value. "Close" is not "correct". 12px is not 10px. #f97316 is not #ff611c.

**CSS specificity check**: Pay special attention to icons inside wrapper components (Button, Link, etc.). Run `getComputedStyle` on the SVG element itself — parent components often override `color` via inheritance or higher-specificity selectors. If the computed color doesn't match what your Tailwind class should produce, use the Tailwind important modifier (prefix the class with !) to force the correct value.

**Step 3: Make a picky mismatch list**

Combine visual issues from Step 1 and numerical mismatches from Step 2 into one list. Include everything — no issue is too small. 2px off? List it. Slightly wrong shade? List it. Font weight 500 vs 600? List it.

**Step 4: Fix everything on the list**

Implement all fixes in code. Do not take screenshots between individual fixes — batch them all.

**Step 5: Repeat from Step 1**

Take another Chrome screenshot. Compare again. If new issues are visible or the numerical audit still shows mismatches, make another list and fix again.

**Exit condition**: The Chrome screenshot matches the Figma reference AND the numerical audit shows zero mismatches. Only then move on.

### Icon & Shape Verification

During any loop iteration, if you spot an icon that looks wrong:
- Call `get_screenshot(iconNodeId, fileKey)` on the Figma icon node
- Screenshot the Chrome icon at 3x zoom
- Compare the SILHOUETTE — stroke count, shape, proportions

NEVER assume an icon library name maps to the Figma icon. Common mismatches:
- "filter" in Figma → often a lines-with-circles icon, NOT a funnel
- "calendar" → many variants (with/without dots, with/without lines)
- "mail" → open envelope vs closed envelope
- "phone" → handset angle and style varies

If an icon doesn't match: try another icon from the library, or extract the SVG from Figma assets and use it inline.

### Anti-Pattern: The "Looks Good Enough" Trap

The biggest failure mode is:
1. Take a full-width screenshot of the component
2. Glance at it
3. Say "looks close" and move on

This ALWAYS misses:
- 2-4px spacing differences
- Wrong icon variant
- Slightly wrong color shade
- Missing shadow or border
- Font weight mismatch (medium vs semibold)

**RULE**: You are not done until the loop exits — zero visual mismatches AND zero numerical mismatches.

## Recovery: Compact & Resume

If you hit the image limit error (or proactively after ~15 screenshots):

1. **Run `/compact`** — this summarizes the conversation and drops all old images from context. Your notes, mismatch lists, and implementation progress are preserved in the compacted summary.
2. **Start fresh visually — retake your reference images:**
   - `get_screenshot` on the Figma layout (overview of the root selection)
   - `get_screenshot` on the Figma detail (the section you're currently working on)
   - Chrome screenshot of the current app state
3. **Continue the refinement loop** from where you left off. You still know everything from the compacted history — what's been implemented, what mismatches were found, what's left to fix. You now have ~17 more image slots.

This is a repeatable cycle:
- Compact → retake 3 reference images → continue refinement loop → compact again if needed
- Even complex components with many refinement cycles can reach perfection this way

## Phase 5: User Review

Once you're satisfied with the result, **stop and ask**:
- "Here's the final result. Are you happy with it?"
- "If something looks off, paste a Figma 'Link to Selection' for the specific area you'd like me to focus on."

If the user provides a new link, re-run Phases 2-4 scoped to that specific selection. This allows targeted refinement of individual building blocks without restarting the full component.

**ULTRA-THINK MODE ENABLED**: Take your time. Perfection over speed.

## Examples

**Good invocation:**
```
/pix
> Paste Figma link: https://figma.com/design/abc123/MyApp?node-id=42-100
```

**What Claude does:**
1. Recon: `get_metadata` + `get_variable_defs` + `get_code_connect_map` (0 images)
2. Figma overview: `get_screenshot` on root (image 1)
3. Study section 1: `get_screenshot` (image 2) + `get_design_context` (text). Memorize every detail.
4. Study section 2: `get_screenshot` (image 3) + `get_design_context` (text). Memorize every detail.
5. Code everything from memory — all sections, all elements (0 images)
6. **Refinement loop, round 1**: Chrome screenshot (image 4) + `getComputedStyle` audit → finds 4 numerical mismatches + 1 icon looks wrong
7. Fix all 4 numerical mismatches. Icon check: `get_screenshot` on Figma icon node (image 5) — wrong icon, swap it.
8. **Refinement loop, round 2**: Chrome screenshot (image 6) + `getComputedStyle` audit → 1 remaining spacing issue
9. Fix spacing.
10. **Refinement loop, round 3**: Chrome screenshot (image 7) + audit → zero mismatches. Pixel-perfect.
11. Done. Total: 7 images

**Bad patterns to avoid:**
- Screenshotting every element individually (budget killer)
- Re-screenshotting the same Figma node (it's static)
- Taking Chrome screenshots after every small CSS tweak
- Using screenshots to verify spacing/color when `getComputedStyle` gives exact numbers
- Not running `/compact` when approaching the image limit
- Calling `get_design_context` on the root selection (token waste)
- Calling `get_variable_defs` more than once (redundant)
- `text-[#f3f3f3]` — hardcoded hex in Tailwind
- `w-[247px]` — magic width number
- Assuming icon color matches text
- Saying "looks close" without numerical verification
