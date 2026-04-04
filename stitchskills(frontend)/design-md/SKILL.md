---
name: design-md
description: Analyze Stitch projects and synthesize a semantic design system into DESIGN.md files
allowed-tools:
  - "stitch*:*"
  - "Read"
  - "Write"
  - "web_fetch"
---

# Stitch DESIGN.md Skill

You are an expert Design Systems Lead. Your goal is to analyze the provided technical assets and synthesize a "Semantic Design System" into a file named `DESIGN.md`.

## Overview

This skill helps you create `DESIGN.md` files that serve as the "source of truth" for prompting Stitch to generate new screens that align perfectly with existing design language. Stitch interprets design through "Visual Descriptions" supported by specific color values.

## Prerequisites

- Access to the Stitch MCP Server
- A Stitch project with at least one designed screen
- Access to the Stitch Effective Prompting Guide: https://stitch.withgoogle.com/docs/learn/prompting/

## The Goal

The `DESIGN.md` file will serve as the "source of truth" for prompting Stitch to generate new screens that align perfectly with the existing design language. Stitch interprets design through "Visual Descriptions" supported by specific color values.

## Retrieval and Networking

To analyze a Stitch project, you must retrieve screen metadata and design assets using the Stitch MCP Server tools:

1. **Namespace discovery**: Run `list_tools` to find the Stitch MCP prefix. Use this prefix (e.g., `mcp_stitch:`) for all subsequent calls.

2. **Project lookup** (if Project ID is not provided):
   - Call `[prefix]:list_projects` with `filter: "view=owned"` to retrieve all user projects
   - Identify the target project by title or URL pattern
   - Extract the Project ID from the `name` field (e.g., `projects/13534454087919359824`)

3. **Screen lookup** (if Screen ID is not provided):
   - Call `[prefix]:list_screens` with the `projectId` (just the numeric ID, not the full path)
   - Review screen titles to identify the target screen (e.g., "Home", "Landing Page")
   - Extract the Screen ID from the screen's `name` field

4. **Metadata fetch**: 
   - Call `[prefix]:get_screen` with both `projectId` and `screenId` (both as numeric IDs only)
   - This returns the complete screen object including:
     - `screenshot.downloadUrl` - Visual reference of the design
     - `htmlCode.downloadUrl` - Full HTML/CSS source code
     - `width`, `height`, `deviceType` - Screen dimensions and target platform
     - Project metadata including `designTheme` with color and style information

5. **Asset download**:
   - Use `web_fetch` or `read_url_content` to download the HTML code from `htmlCode.downloadUrl`
   - Optionally download the screenshot from `screenshot.downloadUrl` for visual reference
   - Parse the HTML to extract Tailwind classes, custom CSS, and component patterns

6. **Project metadata extraction**:
   - Call `[prefix]:get_project` with the project `name` (full path: `projects/{id}`) to get:
     - `designTheme` object with color mode, fonts, roundness, custom colors
     - Project-level design guidelines and descriptions
     - Device type preferences and layout principles

## Analysis & Synthesis Instructions

> ⚠️ **STRICT EXTRACTION RULE**: Every color hex code and font name written into DESIGN.md MUST be copied verbatim from the downloaded HTML/CSS source. Never estimate colors from a screenshot. Never infer fonts from visual appearance. If a value cannot be found in the source code, write `[not found in source]` — do NOT substitute a guess.

### 1. Extract Project Identity (JSON)
- Locate the Project Title
- Locate the specific Project ID (e.g., from the `name` field in the JSON)

### 2. Define the Atmosphere (HTML structure — not screenshot)
Read the HTML/CSS structure (class names, layout patterns, spacing scale) to describe the overall "vibe." Use evocative adjectives (e.g., "Airy," "Dense," "Minimalist," "Utilitarian"). Do NOT derive this from visual inspection of the screenshot alone.

### 3. Map the Color Palette — SOURCE CODE ONLY
**Required process:**
1. Download the HTML source via `htmlCode.downloadUrl` and read it fully
2. Extract every unique color value that appears in the source: inline styles (`color:`, `background-color:`, `border-color:`), Tailwind arbitrary values (`bg-[#hex]`, `text-[#hex]`), and CSS custom properties (`--color-*`)
3. Also extract colors from the `designTheme.customColors` array returned by `get_project`
4. For each color found, record:
   - The **exact hex code as it appears in the source** (do not round, convert, or approximate)
   - A descriptive natural-language name derived from its role in the markup (e.g., "Primary CTA background — #294056")
   - Its functional role based on which elements use it

❌ Do NOT pick colors by eye from the screenshot
❌ Do NOT generate a "representative" palette — only include colors that exist in the source
❌ Do NOT convert hex to HSL/RGB or alter the value in any way

### 4. Extract Typography — SOURCE CODE ONLY
**Required process:**
1. Scan the HTML `<head>` for `<link>` tags pointing to Google Fonts or other font CDNs — copy the exact font family names loaded
2. Search the CSS/Tailwind for `font-family`, `font-sans`, `font-serif`, `font-mono`, or `font-['...']` usages — record exact values
3. Extract `font-size`, `font-weight`, and `letter-spacing` values actually present in the source for headings, body, and labels
4. If the `designTheme` object includes font fields, copy them verbatim

❌ Do NOT invent a font stack because the design "looks like" a sans-serif
❌ Do NOT assume system fonts unless `font-family: system-ui` or similar appears explicitly in the source

### 5. Translate Geometry & Shape (CSS/Tailwind)
Convert technical `border-radius` and layout values into physical descriptions:
- Describe `rounded-full` as "Pill-shaped"
- Describe `rounded-lg` as "Subtly rounded corners"
- Describe `rounded-none` as "Sharp, squared-off edges"

### 6. Describe Depth & Elevation
Explain how the UI handles layers. Describe the presence and quality of shadows (e.g., "Flat," "Whisper-soft diffused shadows," or "Heavy, high-contrast drop shadows"). Base this on actual `box-shadow` / `shadow-*` classes found in the source.

### 7. Extract Visual Effects — SOURCE CODE ONLY
Scan the HTML/CSS for surface and layer effects applied to cards, modals, navbars, hero sections, and any other containers. For each effect found record:

**Glass / Frosted Glass**
- Look for: `backdrop-filter`, `backdrop-blur-*`, `backdrop-blur: blur(Npx)`, `bg-white/10`, `bg-opacity-*`, `saturate-*` combined with blur
- Describe: blur strength (e.g., "Heavy 24px backdrop blur"), tint color + opacity (e.g., "White 10% tint overlay"), border treatment (e.g., "1px solid white/20 edge highlight")

**Gradient Surfaces**
- Look for: `bg-gradient-to-*`, `linear-gradient(...)`, `radial-gradient(...)`, `conic-gradient(...)`
- Record: exact direction, exact color stops with hex codes and opacity values, which elements use them

**Glow & Neon Effects**
- Look for: `shadow-[0_0_Npx_#hex]`, `drop-shadow(...)`, `filter: glow`, `text-shadow`
- Describe: color of the glow (exact hex), spread radius, which element it is applied to

**3D Objects & Transforms**
- Look for: `perspective(...)`, `rotateX(...)`, `rotateY(...)`, `rotateZ(...)`, `transform-style: preserve-3d`, `translate3d(...)`, Three.js/Spline/Lottie `<canvas>` or `<script>` embeds
- Describe: what the 3D element is (e.g., "Rotating product card with 15deg Y-axis tilt on hover"), which section it lives in, and the transform values used

**Noise / Grain Textures**
- Look for: SVG `feTurbulence` filters, `noise.png` or `grain.svg` background images, `mix-blend-mode: overlay` on a texture layer
- Describe: texture type, blend mode, opacity

❌ Do NOT describe effects that do not appear in the source code
❌ Do NOT assume glassmorphism just because cards have a light background

### 8. Extract Text & Component Layout — SOURCE CODE ONLY
Document how content is spatially arranged on the page. Scan the HTML structure and Tailwind layout classes to answer:

**Text Layout**
- Hierarchy: how many heading levels are used (`h1`–`h6` or `text-*xl` scale), and what size/weight each maps to
- Alignment: is body text left-aligned, centered, or mixed per section?
- Max-width constraints: look for `max-w-*` on text containers (e.g., "Body paragraphs constrained to `max-w-2xl` centered")
- Line-height and spacing: `leading-*` classes used on paragraphs vs. headings

**Component Layout**
- Grid systems: `grid-cols-*`, `gap-*`, column counts per section
- Flex patterns: `flex`, `flex-row`, `flex-col`, `items-*`, `justify-*` — describe what each major section uses
- Section stacking: describe the vertical order of sections (e.g., "Hero → Features 3-col grid → Testimonials carousel → CTA banner → Footer")
- Responsive breakpoints: note `sm:`, `md:`, `lg:`, `xl:` overrides that change layout significantly

**UI Component Inventory**
List every distinct UI component found in the source with its styling summary:
- Buttons (primary, secondary, ghost, icon-only)
- Cards (default, featured, interactive)
- Navigation (topbar, sidebar, tab bar, breadcrumb)
- Modals / Drawers / Tooltips
- Badges / Tags / Pills
- Form elements (inputs, selects, checkboxes, toggles)
- Any custom/unique components (e.g., "Pricing toggle slider", "Step progress indicator")

For each, describe: shape, color role, effect applied (if any), and interactive state styling if visible in the source.

### 9. Extract Scroll & Motion Effects — SOURCE CODE ONLY
Scan for animation and scroll-driven behaviour in the HTML/CSS/JS:

**CSS Scroll Effects**
- Look for: `scroll-snap-type`, `scroll-snap-align`, `overflow-y: scroll` with snap containers
- Look for: `@keyframes`, `animation:`, `transition:` declarations tied to scroll position
- Look for: `position: sticky`, `top: *` for sticky headers or floating elements

**JS-driven Scroll Animations**
- Look for: Intersection Observer API usage (`IntersectionObserver`, `data-aos`, `data-sal` attributes)
- Look for: GSAP imports (`gsap`, `ScrollTrigger`), Framer Motion `useScroll`/`useTransform`, Lenis smooth scroll
- Look for: Locomotive Scroll, ScrollReveal, AOS library `<link>` or `<script>` tags
- Describe: which elements animate on scroll, what the animation is (fade-in, slide-up, scale, rotate), and the trigger point

**3D Scroll Effects**
- Look for: `perspective` combined with scroll-linked JS transforms, Three.js scenes that respond to scroll position, Spline scenes with scroll interaction, `parallax` class names or `data-parallax` attributes
- Describe: what moves in 3D as the user scrolls, the axis of movement, and the speed/depth ratio if determinable from the source

**Hover & Micro-interactions**
- Look for: `hover:` Tailwind variants, `transition-*`, `duration-*`, `ease-*` classes, `:hover` CSS rules
- Describe: what changes on hover (color, scale, translate, shadow, border) and the transition speed

## Output Guidelines

- **Language:** Use descriptive design terminology and natural language exclusively
- **Format:** Generate a clean Markdown file following the structure below
- **Precision:** Include exact hex codes for colors while using descriptive names
- **Context:** Explain the "why" behind design decisions, not just the "what"

## Output Format (DESIGN.md Structure)

```markdown
# Design System: [Project Title]
**Project ID:** [Insert Project ID Here]

## 1. Visual Theme & Atmosphere
(Description of the mood, density, and aesthetic philosophy derived from HTML structure and class patterns.)

## 2. Color Palette & Roles
(List only colors extracted verbatim from HTML/CSS source.)
* **[Descriptive Name] ([exact hex])** — [functional role]

## 3. Typography Rules
(Font families copied exactly from <link> tags or font-family declarations. Size/weight scale per heading level and body.)

## 4. Visual Effects
* **Glass / Frosted Glass:** (Which components use it, blur strength, tint color + opacity, border treatment.)
* **Gradients:** (Direction, exact color stops with hex + opacity, which elements.)
* **Glow / Neon:** (Color, spread, which elements.)
* **3D Objects:** (What the element is, transform values, which section, interaction trigger.)
* **Noise / Grain:** (Texture type, blend mode, opacity — or "None detected".)

## 5. Text & Component Layout
### Text Layout
(Heading hierarchy, alignment strategy, max-width constraints, line-height scale.)

### Page Section Structure
(Vertical section order from top to bottom of the page.)

### UI Component Inventory
* **[Component Name]:** (Shape, color role, effects applied, interactive state.)

## 6. Scroll & Motion Effects
* **CSS Scroll Snapping:** (Snap type, which containers — or "None detected".)
* **Scroll-triggered Animations:** (Library used, which elements animate, animation type, trigger point.)
* **3D Scroll Effects:** (What moves in 3D on scroll, axis, depth ratio — or "None detected".)
* **Hover & Micro-interactions:** (What changes on hover per component, transition speed.)

## 7. Layout Principles
(Whitespace strategy, grid systems used, flex patterns, responsive breakpoint behaviour.)
```

## Usage Example

To use this skill for the Furniture Collection project:

1. **Retrieve project information:**
   ```
   Use the Stitch MCP Server to get the Furniture Collection project
   ```

2. **Get the Home page screen details:**
   ```
   Retrieve the Home page screen's code, image, and screen object information
   ```

3. **Reference best practices:**
   ```
   Review the Stitch Effective Prompting Guide at:
   https://stitch.withgoogle.com/docs/learn/prompting/
   ```

4. **Analyze and synthesize:**
   - Extract all relevant design tokens from the screen
   - Translate technical values into descriptive language
   - Organize information according to the DESIGN.md structure

5. **Generate the file:**
   - Create `DESIGN.md` in the project directory
   - Follow the prescribed format exactly
   - Ensure all color codes are accurate
   - Use evocative, designer-friendly language

## Best Practices

- **Be Descriptive:** Avoid generic terms like "blue" or "rounded." Use "Ocean-deep Cerulean (#0077B6)" or "Gently curved edges"
- **Be Functional:** Always explain what each design element is used for
- **Be Consistent:** Use the same terminology throughout the document
- **Be Visual:** Help readers visualize the design through your descriptions
- **Be Precise:** Include exact values (hex codes, pixel values) in parentheses after natural language descriptions

## Tips for Success

1. **Start with the big picture:** Understand the overall aesthetic before diving into details
2. **Look for patterns:** Identify consistent spacing, sizing, and styling patterns
3. **Think semantically:** Name colors by their purpose, not just their appearance
4. **Consider hierarchy:** Document how visual weight and importance are communicated
5. **Reference the guide:** Use language and patterns from the Stitch Effective Prompting Guide

## Common Pitfalls to Avoid

- ❌ Using technical jargon without translation (e.g., "rounded-xl" instead of "generously rounded corners")
- ❌ Omitting color codes or using only descriptive names
- ❌ Forgetting to explain functional roles of design elements
- ❌ Being too vague in atmosphere descriptions
- ❌ Ignoring subtle design details like shadows or spacing patterns
- ❌ **Guessing hex codes from the screenshot** — always extract from HTML/CSS source
- ❌ **Inventing font names** — only write fonts that appear in `<link>` tags, `font-family` declarations, or `designTheme` fields
- ❌ Writing a "typical" or "sensible" color palette when the source gives you the real one
- ❌ Using approximate colors (e.g., "a dark navy, roughly #1a2b3c") — exact values only
- ❌ Assuming glassmorphism or frosted glass just because a card has a light background — look for `backdrop-blur` in source
- ❌ Describing a 3D scroll effect that has no `perspective`, `rotate3d`, or scroll-linked JS in the source
- ❌ Listing a UI component (e.g., "Pricing toggle") that does not appear in the HTML
- ❌ Skipping the scroll/motion section because "it's just a static design" — always scan for `transition`, `animation`, `IntersectionObserver`, and scroll libraries
- ❌ Describing layout from the screenshot rather than reading the actual `grid`, `flex`, and `max-w` classes in the source
