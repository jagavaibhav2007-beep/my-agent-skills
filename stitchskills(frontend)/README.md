# Agent Skills (Frontend)

Custom agent skills for **UI design, prompt engineering, and frontend development**. These skills transform Antigravity into an expert frontend engineer using Google's Stitch and modern UI libraries.

## Available Skills

| Skill | Purpose | When to Use |
|---|---|---|
| 📐 **[design-md](./design-md/SKILL.md)** | Analyze Stitch projects and synthesize a semantic design system into `DESIGN.md` | *"Create a DESIGN.md for my project"* |
| 🪄 **[enhance-prompt](./enhance-prompt/SKILL.md)** | Transform vague UI ideas into polished, Stitch-optimized prompts | *"Enhance this UI prompt"* |
| ⚛️ **[react-components](./react-components/SKILL.md)** | Convert Stitch designs into modular Vite and React components | *"Convert this design to React"* |
| 🎬 **[remotion](./remotion/SKILL.md)** | Generate walkthrough videos from Stitch projects using Remotion | *"Show me a walkthrough of my screens"* |
| 🎨 **[shadcn-ui](./shadcn-ui/SKILL.md)** | Expert guidance for integrating and building applications with shadcn/ui | *"How do I add a shadcn sidebar?"* |
| 🔄 **[stitch-loop](./stitch-loop/SKILL.md)** | Iteratively build websites using an autonomous baton-passing loop | *"Start the build loop"* |
| 🎭 **[motion-design](./motion-design/SKILL.md)** | Add production-grade Framer Motion + GSAP animations (spring physics, scroll triggers, stagger lists, shared layouts, Lenis smooth scroll, SplitType text reveals) | *"Make it feel alive"* |
| 📜 **[scroll-storytelling](./scroll-storytelling/SKILL.md)** | Build immersive scroll-driven narratives — Lenis + GSAP ScrollTrigger, pinned sections, horizontal scroll, parallax layers, text reveals, progress bars | *"Make it scroll like an Apple page"* |
| 🧊 **[three-d](./three-d/SKILL.md)** | Build 3D web experiences with React Three Fiber + Drei — 3D hero backgrounds, particle systems, scroll-synced 3D, Spline embeds, WebGL optimization | *"Add a 3D background"* |
| 🔀 **[state-flow](./state-flow/SKILL.md)** | Architect server-state (TanStack Query v5) and client-state (Zustand v5) using the Iron Rule, slice pattern, optimistic updates, and query key registry | *"Connect this to real data"* |

## The Frontend Workflow

```
┌─────────────────────────────────────────────────────────┐
│  Phase 1: Design & Ideation                             │
│                                                         │
│  "Enhance this UI prompt" → Enhance Prompt              │
│  "Generate screen" → Stitch (through Antigravity)      │
│  "Create DESIGN.md" → Design-MD                         │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  Phase 2: Build & Iterate                               │
│                                                         │
│  "Convert to React" → React-Components                  │
│  "Add shadcn sidebar" → Shadcn-UI                       │
│  "Start build loop" → Stitch-Loop                       │
│  "Make it feel alive" → Motion-Design                   │
│  "Make it scroll like Apple" → Scroll-Storytelling      │
│  "Add a 3D background" → Three-D                        │
│  "Connect this to real data" → State-Flow               │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  Phase 3: Demo & Showcase                               │
│                                                         │
│  "Generate walkthrough" → Remotion                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Folder Structure

```
stitchskills(frontend)/
├── README.md                    ← You are here
├── design-md/
│   └── SKILL.md                 ← Design system synthesis
├── enhance-prompt/
│   └── SKILL.md                 ← Prompt engineering logic
├── react-components/
│   └── SKILL.md                 ← Design-to-code transformation
├── remotion/
│   └── SKILL.md                 ← Walkthrough video generation
├── shadcn-ui/
│   └── SKILL.md                 ← Component integration guide
├── stitch-loop/
│   └── SKILL.md                 ← Autonomous iteration loop
├── motion-design/
│   └── SKILL.md                 ← Framer Motion + GSAP + Lenis + SplitType
├── scroll-storytelling/
│   └── SKILL.md                 ← Lenis, pinned sections, parallax, text reveals
├── three-d/
│   └── SKILL.md                 ← React Three Fiber + Drei + Spline
└── state-flow/
    └── SKILL.md                 ← TanStack Query + Zustand architecture
```

## Companion Skills

- **[agentskills(backend)](../agentskills(backend)/)** — Logic, database, and security
- **[agentskills(product)](../agentskills(product)/)** — Planning and research
