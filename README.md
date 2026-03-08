# 🛡️ MyAgents-AI-Arsenal

A comprehensive suite of specialized AI agent skills designed for **Antigravity**. These skills transform your AI assistant into an expert product manager, frontend designer, and backend engineer.

## 🚀 Key Features

- **Product Strategy:** Planning, research, and critique from real user perspectives.
- **Frontend Mastery:** Design system synthesis, prompt engineering, and design-to-code for React.
- **Backend Precision:** Debugging, security auditing, database management, and refactoring.
- **AI Memory:** Persistent context management to prevent repetitive mistakes and smooth session handovers.

## 📂 Repository Structure

```
MyAgents-AI-Arsenal/
├── agentskills(product)/        ← Strategy, planning, and critique
│   ├── build-planner/
│   ├── feature-architect/
│   └── product-critic/
├── stitchskills(frontend)/      ← UI design, prompt engineering, and React
│   ├── design-md/
│   ├── enhance-prompt/
│   ├── react-components/
│   ├── remotion/
│   ├── shadcn-ui/
│   └── stitch-loop/
├── agentskills(backend)/        ← Debugging, security, and logic
│   ├── database/
│   ├── debugger/
│   ├── docs-memory/
│   ├── refactorer/
│   ├── security/
│   └── testing/
└── .github/workflows/           ← CI/CD for skill validation
```

## 🛠️ Getting Started

### 1. Integration
Copy the desired skill folder into your Antigravity project's `agentskills` or equivalent directory.

### 2. Activation
Simply ask Antigravity to perform a task related to any of the skills:
- *"GSD this"* (Build Planner)
- *"Critique my app"* (Product Critic)
- *"Run a security audit"* (Security Agent)
- *"Convert this design to React"* (React Components)

## 🧠 Inspired By
These skills are modeled after top-tier open-source projects including:
- **Product:** GSD, Gemini/Claude prompting guides.
- **Frontend:** Google Stitch, Shadcn/UI, Remotion.
- **Backend:** SWE-agent, LLM Guard, LlamaIndex, Mem0, Cognee, Refact.ai.

## 🔐 Security Note
**Never hardcode API keys.** Use environment variables for all sensitive configuration.
