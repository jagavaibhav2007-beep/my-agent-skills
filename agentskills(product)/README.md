# Agent Skills (Product)

Custom agent skills for **product design, research, strategy, and build planning**. These skills make Antigravity think like a product manager, UX researcher, and project planner.

## Available Skills

| Skill | Purpose | When to Use |
|---|---|---|
| 🔍 **[product-critic](./product-critic/SKILL.md)** | Critique your app from real user perspectives, research competitors, deliver a findings report | *"Critique my app — what would real users think?"* |
| 🏗️ **[feature-architect](./feature-architect/SKILL.md)** | Read the critique report, pick ONE best-fit feature, produce an implementation plan | *"Pick a feature from the report and plan it"* |
| 📋 **[build-planner](./build-planner/SKILL.md)** | GSD-inspired planner that breaks your project into AI-optimized phases and produces a PLAN.md | *"Plan what I want to build"* or *"GSD this"* |

## The Full Pipeline

These skills can work as a pipeline or standalone:

```
┌─────────────────────────────────────────────────────────┐
│  OPTION A: Build something new                          │
│                                                         │
│  "I want to build [idea]"                               │
│       → Build Planner → PLAN.md                         │
│       → Execute phase by phase                          │
│       → Product Critic → Review what was built          │
│       → Feature Architect → Pick next feature           │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  OPTION B: Improve something existing                   │
│                                                         │
│  "Critique my app"                                      │
│       → Product Critic → Research Report                │
│       → Feature Architect → Pick ONE + Plan it          │
│       → Build it using backend/frontend skills          │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  OPTION C: Standalone                                   │
│                                                         │
│  Each skill also works independently                    │
│  whenever you need just that one thing.                 │
└─────────────────────────────────────────────────────────┘
```

## What Makes Each Skill Special

| Skill | Inspired By | Key Technique |
|---|---|---|
| Product Critic | Nielsen Norman Group, UX Research | 5 user personas + Nielsen's 10 heuristics + competitive research |
| Feature Architect | Architecture-fit scoring | Scores features on infra reuse, MCP leverage, pattern match |
| Build Planner | [GSD](https://github.com/gsd-build/get-shit-done), [Gemini prompting guide](https://ai.google.dev/gemini-api/docs/prompting-strategies), [Claude prompting guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct) | AI-optimized task format with verification, following official Gemini XML templates and Claude's clarity principles |

## Folder Structure

```
agentskills(product)/
├── README.md                    ← You are here
├── product-critic/
│   └── SKILL.md                 ← Research & critique report
├── feature-architect/
│   └── SKILL.md                 ← Architecture-fit feature planning
└── build-planner/
    └── SKILL.md                 ← GSD-inspired phased build planning
```

## Companion Skills

- **[agentskills(backend)](../agentskills(backend)/)** — Debugging, security, and database (used during build execution)
- **[stitchskills(frontend)](../stitchskills(frontend)/)** — UI design and frontend (used during build execution)
