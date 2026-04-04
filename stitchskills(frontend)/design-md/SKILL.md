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

5a. **Multi-screen retrieval** (if `list_screens` returns more than one screen):
   - Retrieve the HTML source for the first 3 screens by order (or all screens if fewer than 3 exist)
   - During color, typography, and component extraction: **only record tokens that appear in at least 2 screens**
   - Tokens that appear in only one screen are project-specific overrides — note them separately under "Screen-specific overrides" in the output, not in the main design system
   - This ensures the DESIGN.md reflects the actual design system, not one-off page styles

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
Read the HTML/CSS structure and use the signals below to determine the vibe. Do NOT derive this from visual inspection of the screenshot alone.

| Signal found in source | Atmosphere indicator |
|---|---|
| Large `py-24`, `py-32`, sparse `gap-*`, few elements per section | Airy / Spacious |
| Tight `gap-2`, `grid-cols-4+`, many elements per section | Dense / Information-rich |
| `text-xs`, `tracking-widest`, `uppercase`, muted palette | Refined / Editorial |
| `font-black`, large `text-7xl+`, saturated colors | Bold / Expressive |
| `rounded-none`, `border`, monochrome palette | Utilitarian / Minimal |
| `backdrop-blur`, `bg-white/10`, dark background | Glassmorphic / Futuristic |
| `shadow-lg+`, strong contrast, `font-bold` headings | Grounded / Corporate |

Pick the 2–3 indicators with the most evidence in the source and combine them (e.g., "Airy and Refined").

### 3. Map the Color Palette — SOURCE CODE ONLY
**Required process:**
1. Download the HTML source via `htmlCode.downloadUrl` and read it fully
2. Extract every unique color value that appears in the source: inline styles (`color:`, `background-color:`, `border-color:`), Tailwind arbitrary values (`bg-[#hex]`, `text-[#hex]`), and CSS custom properties (`--color-*`)
3. Also extract colors from the `designTheme.customColors` array returned by `get_project`
4. For each color found, record:
   - The **exact hex code as it appears in the source** (do not round, convert, or approximate)
   - A descriptive natural-language name derived from its role in the markup (e.g., "Primary CTA background — #294056")
   - Its functional role based on which elements use it

**Tailwind named color resolution:**
If the source uses standard Tailwind color classes (e.g., `bg-blue-600`, `text-gray-900`, `bg-slate-800`) with no arbitrary hex override, resolve them to their Tailwind v3 default hex values using this reference and mark them as `[resolved from Tailwind]` — not `[not found in source]`:

| Class | Hex |
|---|---|
| `slate-800` | #1e293b |
| `slate-900` | #0f172a |
| `gray-900` | #111827 |
| `gray-100` | #f3f4f6 |
| `zinc-900` | #18181b |
| `neutral-900` | #171717 |
| `blue-600` | #2563eb |
| `blue-500` | #3b82f6 |
| `indigo-600` | #4f46e5 |
| `violet-600` | #7c3aed |
| `purple-600` | #9333ea |
| `pink-600` | #db2777 |
| `rose-500` | #f43f5e |
| `red-600` | #dc2626 |
| `orange-500` | #f97316 |
| `amber-400` | #fbbf24 |
| `yellow-400` | #facc15 |
| `green-600` | #16a34a |
| `emerald-500` | #10b981 |
| `teal-600` | #0d9488 |
| `cyan-500` | #06b6d4 |
| `white` | #ffffff |
| `black` | #000000 |

For any Tailwind shade not listed above, look up the exact hex in the Tailwind v3 default palette — do not guess.

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
List every distinct UI component found in the source. Count how many times each appears across all retrieved screens, then rank them:

- **Primary components** (appear 3+ times or on every screen) — these carry the brand identity; describe in full detail
- **Secondary components** (appear 1–2 times) — describe more briefly

Categories to scan for:
- Buttons (primary, secondary, ghost, icon-only)
- Cards (default, featured, interactive)
- Navigation (topbar, sidebar, tab bar, breadcrumb)
- Modals / Drawers / Tooltips
- Badges / Tags / Pills
- Form elements (inputs, selects, checkboxes, toggles)
- Any custom/unique components (e.g., "Pricing toggle slider", "Step progress indicator")

For each **primary** component describe: shape, color role, effect applied (if any), interactive state styling, and frequency count.
For each **secondary** component describe: shape, color role, frequency count.

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

> **Handoff note:** This step documents *what* motion exists. It does not implement it. When generating a new screen that needs to replicate or extend these scroll and motion effects, pass the Scroll & Motion Effects section of this DESIGN.md to the `motion-design` skill, which handles Framer Motion and GSAP implementation.

## Post-Write Verification

After writing DESIGN.md, perform this verification pass before finishing:

1. **Color check** — for every hex code written in DESIGN.md, confirm it appears verbatim in the downloaded HTML source or in the `designTheme.customColors` array, or is marked `[resolved from Tailwind]`. If any hex code cannot be traced to one of these sources, replace it with `[not found in source]` and flag it in a comment above the DESIGN.md.

2. **Font check** — for every font family name written in DESIGN.md, confirm it appears in a `<link>` tag, `font-family` declaration, or `designTheme` font field. If not, mark it `[not found in source]`.

3. **Component check** — for every component listed in the UI Component Inventory, confirm it exists in at least one retrieved screen's HTML. Remove any that do not.

4. **Effect check** — for every visual effect described (glass, gradient, glow, 3D, noise), confirm the specific CSS property or class that triggers it was found in the source. If the specific property cannot be cited, remove the effect from the output.

Only mark the task complete after this verification passes.

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

## Before / After: What Good vs Bad Output Looks Like

Use these examples to recognise a correct extraction vs a hallucinated one.

### Color Palette — BAD (hallucinated)
```markdown
## 2. Color Palette & Roles
* **Deep Ocean Navy (#1a2b4c)** — Primary brand color used for headers
* **Warm Ivory (#f5f0e8)** — Background tone giving the design warmth
* **Accent Gold (#d4a843)** — Used to highlight key CTAs
```
Why it's bad: None of these hex codes were in the source. The model constructed a "plausible-looking" palette from the screenshot.

### Color Palette — GOOD (extracted)
```markdown
## 2. Color Palette & Roles
* **Dark Slate background (#0f172a) [resolved from Tailwind: bg-slate-900]** — Page background applied to `<body>` and all full-bleed sections
* **Electric Indigo (#4f46e5) [resolved from Tailwind: bg-indigo-600]** — Primary CTA button background and active nav indicator
* **Frosted white overlay (rgba(255,255,255,0.08))** — Card surface tint, extracted from inline style on `.card` elements
* **Border mist (#ffffff1a)** — 1px card border, from `border-white/10` Tailwind class
```
Why it's good: Every value is sourced. Tailwind resolutions are labelled. rgba values are copied verbatim.

---

### Typography — BAD (hallucinated)
```markdown
## 3. Typography Rules
Font: Inter (clean, modern sans-serif). Headers use bold weight, body uses regular.
```
Why it's bad: "Inter" was not in a `<link>` tag or `font-family` declaration — the model assumed it because Inter is common.

### Typography — GOOD (extracted)
```markdown
## 3. Typography Rules
* **Primary font: "Geist", sans-serif** — loaded via `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Geist">` in `<head>`
* **Monospace font: "Geist Mono"** — loaded via same stylesheet, used on code labels (`font-mono` class)
* **h1:** `text-6xl font-bold tracking-tight` → 60px, 700 weight, tight tracking
* **h2:** `text-3xl font-semibold` → 30px, 600 weight
* **Body:** `text-base leading-7` → 16px, 28px line-height
* **Label/Caption:** `text-sm text-slate-400` → 14px, muted slate
```
Why it's good: Font names match the actual `<link>` tag. Every size/weight is a Tailwind class found in the source.

---

### Visual Effects — BAD (hallucinated)
```markdown
## 4. Visual Effects
* **Glass / Frosted Glass:** Cards use a frosted glass effect with soft blur and white tint
```
Why it's bad: No CSS evidence cited. The model assumed glass because the cards looked light.

### Visual Effects — GOOD (extracted)
```markdown
## 4. Visual Effects
* **Glass / Frosted Glass:** Feature cards use `backdrop-blur-xl` (24px blur) + `bg-white/8` (white 8% tint) + `border border-white/10` (10% white edge highlight). Found on `.feature-card` divs in the Hero and Features sections.
* **Gradients:** Hero section background uses `bg-gradient-to-br from-slate-900 via-indigo-950 to-slate-900`. CTA button uses `bg-gradient-to-r from-indigo-600 to-violet-600`.
* **Glow / Neon:** CTA button has `shadow-[0_0_40px_rgba(99,102,241,0.4)]` — indigo glow at 40px spread.
* **3D Objects:** None detected — no `perspective`, `rotateX/Y`, or canvas embeds found in source.
* **Noise / Grain:** None detected.
```

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
