# Agent Skills (Backend)

Custom agent skills that define **how Antigravity behaves** when performing backend tasks. Each skill is modeled after real-world open-source AI agent repositories for maximum effectiveness.

## Available Skills

| Skill | Inspired By | Purpose |
|---|---|---|
| 🐛 **[debugger](./debugger/SKILL.md)** | [SWE-agent](https://github.com/princeton-nlp/SWE-agent), [Kaizen Agent](https://github.com/Kaizen-agent/kaizen-agent), [Blinky](https://github.com/blinkydotdev/blinky) | Reactive bug fixing + proactive codebase scanning across 7 categories |
| 🔒 **[security](./security/SKILL.md)** | [LLM Guard](https://github.com/protectai/llm-guard), [vuln-agent](https://github.com/samuelberston/vuln-agent), [NVISO](https://github.com/NVISOsecurity/cyber-security-llm-agents) | Bidirectional scanner pipeline, confidence-scored triage, DDoS/rate-limit defense |
| 🗄️ **[database](./database/SKILL.md)** | [LlamaIndex](https://github.com/run-llama/llama_index), [db-ally](https://github.com/deepsense-ai/db-ally), [OSADA](https://github.com/spron-in/osada) | Structured views (SQL injection impossible by design), text-to-SQL, vector DB |
| 🧹 **[refactorer](./refactorer/SKILL.md)** | [Refact.ai](https://github.com/smallcloudai/refact) (Apache-2.0), [Martin Fowler's Catalog](https://refactoring.com/catalog/), [Refactoring.guru](https://refactoring.guru) | Detect code smells, apply proven refactoring techniques, clean up vibe-coded spaghetti |
| 🧪 **[testing](./testing/SKILL.md)** | [Qodo Cover](https://github.com/qodo-ai/qodo-cover) (AGPL-3.0), [CoverUp](https://github.com/plasma-umass/coverup) (Apache-2.0), [EvoMaster](https://github.com/WebFuzzing/EvoMaster) (Apache-2.0) | Generate unit/integration/API tests for existing code, iterative coverage loop |
| 🧠 **[docs-memory](./docs-memory/SKILL.md)** | [Auto Codebase Documenter](https://github.com/abryant710/auto-codebase-documenter) (MIT), [Mem0](https://github.com/mem0ai/mem0) (Apache-2.0), [Cognee](https://github.com/topoteretes/cognee) (Apache-2.0) | Auto-generate project docs + maintain AI memory so the agent never repeats mistakes |
| 🤖 **[ai-engineer](./ai-engineer/SKILL.md)** | [LangChain](https://github.com/langchain-ai/langchain), [LlamaIndex](https://github.com/run-llama/llama_index), [CrewAI](https://github.com/crewAIInc/crewAI) | Build RAG systems, AI agents, and production-scale AI applications |
| 🚀 **[deployment-engineer](./deployment-engineer/SKILL.md)** | [Vercel](https://vercel.com/docs), [Railway](https://docs.railway.app), [Fly.io](https://fly.io/docs), [actions/starter-workflows](https://github.com/actions/starter-workflows) | Detect stack → pick platform → pre-deploy checklist → env var audit → CI/CD wiring → failure diagnosis |

## Key Techniques Learned from Each Repo

### From SWE-agent (Debugger)
- **Reproduce first, fix second** — Always create a reproduction script before writing a fix
- **Structured codebase navigation** — Follow the stack trace methodically
- **Submit protocol** — Verify → clean up → submit only when all tests pass

### From LLM Guard (Security)
- **Bidirectional scanning** — Scan BOTH inputs AND outputs through specialized scanners
- **Anonymize/Deanonymize vault** — Replace PII with tokens before processing
- **Scanner pipeline pattern** — Each scanner can modify, block, or pass through

### From db-ally (Database)
- **Structured Views + IQL** — Abstraction layer making SQL injection impossible by design
- **AI selects from predefined filter methods** — Cannot generate arbitrary SQL

### From Refact.ai (Refactorer)
- **Context-aware RAG** — Understand the full codebase before touching any code
- **Small, verified steps** — One refactoring per step, verify between each
- **Match existing patterns** — Don't introduce new conventions, follow what exists

### From Martin Fowler's Catalog (Refactorer)
- **5 categories of code smells** — Bloaters, Change Preventers, Dispensables, Couplers, OO Abusers
- **60+ named refactoring techniques** — Extract Function, Guard Clauses, Move Method, etc.
- **Vibe-coding-specific smells** — Framework Soup, Prompt Layers, God Component

### From Qodo Cover + CoverUp (Testing)
- **Coverage-guided iterative loop** — Generate → run → measure coverage → generate more for uncovered lines → repeat
- **4-component pipeline** — Prompt Builder → AI Caller → Test Runner → Coverage Parser (from Qodo Cover)
- **Test quality over quantity** — FIRST principles: Fast, Independent, Repeatable, Self-validating, Timely

### From EvoMaster (Testing)
- **Evolutionary API fuzzing** — Test APIs with valid inputs, missing fields, invalid types, auth failures
- **Black-box endpoint testing** — Test HTTP endpoints without knowing internals
- **System-level test generation** — Cover REST, GraphQL, and RPC APIs

### From Mem0 + Cognee + Claude Code (Docs & Memory)
- **Multi-level memory** — User, session, and project state persisted across sessions (from Mem0)
- **Knowledge graph connections** — Link bugs to fixes, decisions to consequences (from Cognee)
- **Markdown-first persistence** — Plain .md files that are human-readable, Git-versionable, auto-loaded by AI (from Claude Code/OpenClaw)
- **File-by-file documentation** — Walk every source file and generate comprehensive docs mirroring the project structure (from Auto Codebase Documenter)

### From LangChain, LlamaIndex, CrewAI (AI Engineer)
- **Advanced RAG Patterns** — Chunking strategies, hybrid search, semantic caching, and quality gates
- **Multi-Agent Orchestration** — Role-based agents, structured task delegation, and stateful graphs
- **Production Observability** — The MELT framework, rate limiting, retries, and Guardrails

### From Vercel, Railway, Fly.io docs + GitHub Actions starter-workflows (Deployment Engineer)
- **Read before recommending** — Stack detection from package.json + existing config files before any platform decision
- **Platform decision tree** — Next.js→Vercel, Express/Node→Railway, containerized→Fly.io
- **Pre-deploy gate** — Build passes, tests pass, no secrets in source, .env gitignored, env vars documented
- **Failure diagnosis** — 6 pattern-matched failure categories (build, runtime, port, env, health check, Vercel-specific)

## How to Activate a Skill

Simply ask Antigravity to perform a task related to the skill:

- **Debugger:** *"Debug why the login crashes"* or *"Scan my code for bugs"*
- **Security:** *"Run a security audit"* or *"Check for hardcoded secrets"*
- **Database:** *"Design the schema for my app"* or *"Set up Supabase tables with RLS"*
- **Refactorer:** *"Clean up this file"* or *"Refactor my codebase"*
- **Testing:** *"Write tests for this"* or *"Generate tests for what I built"*
- **Docs & Memory:** *"Document my project"* or *"Update the memory"*
- **AI Engineer:** *"Add AI to my app"* or *"Build a RAG system"*
- **Deployment Engineer:** *"Deploy my app"* or *"Set up CI/CD"* or *"It works locally but not in production"*

## Folder Structure

```
agentskills(backend)/
├── README.md              ← You are here
├── debugger/
│   └── SKILL.md           ← SWE-agent + Kaizen + Blinky
├── security/
│   └── SKILL.md           ← LLM Guard + vuln-agent + NVISO
├── database/
│   └── SKILL.md           ← LlamaIndex + db-ally + OSADA
├── refactorer/
│   └── SKILL.md           ← Refact.ai + Fowler + Refactoring.guru
├── testing/
│   └── SKILL.md           ← Qodo Cover + CoverUp + EvoMaster
├── docs-memory/
│   └── SKILL.md           ← Auto Documenter + Mem0 + Cognee + Claude Code
├── ai-engineer/
│   └── SKILL.md           ← LangChain + LlamaIndex + CrewAI
└── deployment-engineer/
    └── SKILL.md           ← Vercel + Railway + Fly.io + GitHub Actions
```

## Companion Skills

- **[agentskills(product)](../agentskills(product)/)** — Product critique, feature planning, and build planning
- **[stitchskills(frontend)](../stitchskills(frontend)/)** — UI design and frontend skills
