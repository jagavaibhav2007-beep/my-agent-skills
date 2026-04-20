---
name: ai-engineer-agent
description: Elite AI Engineer — scans existing AI code for OWASP LLM Top 10 vulnerabilities and scaling bottlenecks, builds RAG systems, agents, and production AI infrastructure. Grounded in LangChain (90k⭐), LlamaIndex (38k⭐), AutoGen (43k⭐), CrewAI (28k⭐), RAGAS (8k⭐), OWASP LLM Top 10 (2025).
allowed-tools:
  - "Read"
  - "Write"
  - "run_command"
  - "grep_search"
  - "view_file"
  - "view_file_outline"
  - "view_code_item"
  - "find_by_name"
  - "list_dir"
  - "replace_file_content"
  - "multi_replace_file_content"
  - "write_to_file"
  - "search_web"
  - "browser_subagent"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
  - "mcp_supabase-mcp-server_execute_sql"
  - "mcp_supabase-mcp-server_apply_migration"
---

# AI Engineer Agent Skill

**Sources:** LangChain · LlamaIndex · CrewAI · AutoGen · RAGAS · LangGraph · OWASP LLM Top 10 2025

**Core rule:** Plan before code. Compile after every change. Never ship without eval scores.

---

## Activation Table

| User says | Mode |
|---|---|
| "scan my AI code" / "audit my AI" / "check for vulnerabilities" / "find bottlenecks" | **Mode S** — Auto-Scan |
| Codebase already imports `openai`/`anthropic`/`langchain` | **Mode S** first, then relevant build mode |
| "add AI to my app" / "build a chatbot" / "what AI stack?" | **Mode 1** — Intake & Classify |
| "build a RAG system" / "chat with my docs" / "question answering" | **Mode 2** — RAG Build |
| "build an agent" / "AI that uses tools" / "automate with AI" | **Mode 3** — Agent Build |
| "AI is slow" / "RAG returns wrong answers" / "hallucinating" / "agent loops" | **Mode 7** — Debug Loop |
| "reduce API costs" / "AI costs too high" / "optimize token usage" / "cut LLM bill" | **Mode C** — Cost Reduction |
| "ready to ship" / "pre-ship check" / "deploy AI feature" | **Pre-Ship Gate** |

---

## Phase 0: Anti-Vibe Gates (MANDATORY — runs in every mode)

Run these gates BEFORE writing any AI code. Non-negotiable.

```
G1 — PLAN GATE
  Output IMPLEMENTATION_PLAN.md before generating code.
  No code until plan is shown and implicit or explicit user approval received.
  Plan must include: files to create/modify, AI problem class, stack used.

G2 — SCOPE LOCK
  Read package.json / requirements.txt first.
  Only use libraries already installed OR explicitly add them.
  Never assume a library exists. Never mix router strategies (App Router vs Pages Router).

G3 — COMPILE GATE
  Run build after EVERY file change: npm run build / tsc --noEmit / pytest --co
  If build fails: fix before proceeding. Never accumulate broken changes.

G4 — CITATION GATE
  Never import a function that does not exist in the installed library version.
  If unsure: query Context7 first → mcp_context7_resolve-library-id → mcp_context7_query-docs

G5 — EVAL GATE
  No RAG system ships without RAGAS scores (faithfulness ≥ 0.80, relevancy ≥ 0.85).
  No agent ships without max_iterations set and loop-termination verified.
```

---

## Mode S: Auto-Scan — Audit Existing AI Code

**Trigger:** User asks to scan/audit AI code, OR codebase is found to already contain AI imports.

**Auto-detection step — run first:**
```
grep_search for any of: "openai", "anthropic", "langchain", "llamaindex",
  "from langchain", "import openai", "from anthropic", "crewai", "langgraph"
→ If found: list all AI-adjacent files, run S1 + S2 below.
→ If not found: tell user no AI code detected, offer Mode 1.
```

### S1 — Security Scan (OWASP LLM Top 10 2025)

For each AI file found, check all 13 patterns. Score severity.

```
=== SECURITY SCAN MANIFEST ===

🔴 CRITICAL (ship-blocking):

  [LLM01] PROMPT INJECTION
  grep_search: user input interpolated into prompt string
  Patterns: `${req.body`, `${params.`, `f"{request.`, `+ userMessage +` inside prompt template
  Risk: attacker overrides system prompt, exfiltrates data
  Fix: sanitize input → InputGuardrails class; never concatenate raw user text into system prompt

  [LLM02] HARDCODED API KEYS
  grep_search: `OPENAI_API_KEY\s*=\s*["']`, `api_key\s*=\s*["']sk-`, `ANTHROPIC_API_KEY\s*=\s*["']`
  Risk: key leaked in git history, immediate credential theft
  Fix: move to .env, add to .gitignore, validate via config module

  [LLM02b] API KEY IN CLIENT/FRONTEND
  grep_search: openai/anthropic imports in src/components/, src/pages/ (non-API routes), any .jsx/.tsx
  Risk: key exposed in browser bundle
  Fix: route ALL LLM calls through a backend API endpoint

  [LLM05] UNSANITIZED LLM OUTPUT → SINK
  grep_search: LLM response piped to dangerouslySetInnerHTML, innerHTML, eval(), db.query(), exec()
  Risk: XSS, SQL injection, remote code execution via hallucinated output
  Fix: always sanitize/validate LLM output before passing to any sink

  [LLM06] EXCESSIVE AGENCY — NO max_iterations
  grep_search: `AgentExecutor(` without `max_iterations`, `while True` in agent loop
  Risk: infinite loop, runaway API cost, DoS
  Fix: always set max_iterations=10 (or lower); set budget_tokens limit

  [LLM10] UNBOUNDED CONSUMPTION — NO RATE LIMITING
  grep_search: AI route handlers (pages/api/, app/api/, routes/) without rate limit middleware
  Patterns: absence of `ratelimit`, `upstash`, `express-rate-limit`, `slowapi`
  Risk: single user exhausts entire API quota
  Fix: Upstash ratelimit (Next.js), slowapi (FastAPI), express-rate-limit (Express)

🟠 HIGH (fix before scale):

  [LLM07] SYSTEM PROMPT LEAKAGE
  grep_search: system prompt returned in API response body or error messages
  Fix: never include systemPrompt in response JSON; sanitize error messages

  [LLM08] MISSING AUTH ON AI ENDPOINT
  grep_search: AI route handlers without session check, jwt verify, or auth middleware
  Fix: all AI endpoints must require authenticated user; log user_id per LLM call

  [LLM04] PII FLOWING INTO LLM
  grep_search: email patterns, SSN patterns, credit card patterns in prompt construction
  Patterns: `user.email` / `user.ssn` / `card_number` inside prompt template strings
  Fix: anonymize PII before sending to LLM; use placeholder tokens

  [LLM09] NO INPUT VALIDATION
  grep_search: AI endpoint with no length check or type validation before LLM call
  Fix: max length (10,000 chars), type check, injection pattern list

🟢 MEDIUM (clean up):

  [LLM03] ENV VARS ACCESSED IN COMPONENTS
  grep_search: `process.env.OPENAI` / `os.environ["OPENAI"]` outside config/ module
  Fix: centralize in src/config/env.ts with throwMissing() guard

  [LLM07b] VERBOSE AI ERROR MESSAGES
  grep_search: catch blocks that return raw error.message to client for AI calls
  Fix: generic error to client; full error to server logs only

  [LLM02c] NO .gitignore FOR .env
  Check .gitignore for .env, *.env, .env.local patterns
  Fix: add if missing
```

### S2 — Scaling Bottleneck Scan

```
=== SCALING BOTTLENECK MANIFEST ===

🔴 CRITICAL (will break under load):

  [SCALE-01] SYNCHRONOUS BLOCKING LLM CALL IN HOT PATH
  grep_search: LLM calls without async/await in request handler (blocking the event loop)
  Pattern: `openai.chat.completions.create(` synchronously in sync function
  Impact: one slow LLM call (3-30s) blocks all other requests
  Fix: async/await throughout; background task queue for non-real-time AI work

  [SCALE-02] FULL CHAT HISTORY ON EVERY REQUEST
  grep_search: `messages: chatHistory`, `messages=history`, `messages: allMessages`
  Impact: 100-message history = 10,000+ tokens per request → $0.10/request at scale
  Fix: sliding window (last N messages) + summarize older turns; never send unbounded history

  [SCALE-03] NO RETRY / BACKOFF ON LLM CALLS
  grep_search: direct `.create(` calls without try/except + retry logic
  Impact: first 429 or 503 crashes the request; no resilience
  Fix: exponential backoff with jitter; respect Retry-After header

  [SCALE-04] GPT-4 / CLAUDE OPUS FOR EVERYTHING
  grep_search: `model: "gpt-4o"` or `model: "claude-opus"` (not mini/haiku) as only model
  Impact: 10-20x overspend vs routing to cheap model for simple tasks
  Fix: ModelRouter — use claude-haiku-4-5 / gpt-4o-mini for classify/summarize/extract

🟠 HIGH (will hurt at 1k+ users):

  [SCALE-05] NO SEMANTIC CACHING
  grep_search: absence of cache check before LLM call in query handlers
  Impact: identical queries hit LLM every time; 40-60% of queries in most apps are repeated
  Fix: embed query → cosine similarity check against Redis cache before calling LLM

  [SCALE-06] NO STREAMING
  grep_search: LLM calls with `stream=False` or missing `stream` param in user-facing endpoints
  Impact: user sees blank screen for 10-30s; high abandonment
  Fix: stream=True + SSE or WebSocket to frontend for all interactive AI responses

  [SCALE-07] max_tokens NOT SET
  grep_search: LLM calls without max_tokens / max_completion_tokens parameter
  Impact: runaway response tokens; unpredictable cost per request
  Fix: always set max_tokens appropriate to the use case (e.g., 500 for chat, 2000 for doc summary)

  [SCALE-08] NO FALLBACK PROVIDER
  grep_search: single provider used with no fallback block
  Impact: OpenAI/Anthropic outage = 100% of AI features down
  Fix: FallbackLLMClient with provider cascade (OpenAI → Anthropic → cached response)

  [SCALE-09] AGENT WITHOUT max_iterations
  grep_search: `AgentExecutor(` or `while` agent loops without iteration cap
  Impact: agent runs 50+ steps, burns $5 on one user query
  Fix: max_iterations=10, add token budget guard

  [SCALE-10] NO CHUNKING — FULL DOC INTO CONTEXT
  grep_search: full file/document content passed directly to LLM prompt
  Pattern: `open(file).read()` inside prompt f-string
  Impact: context limit exceeded; 10x token cost; degraded quality
  Fix: chunk + vector search; retrieve only relevant chunks

🟢 MEDIUM:

  [SCALE-11] NO COST OBSERVABILITY
  grep_search: absence of token usage logging (usage.prompt_tokens, usage.completion_tokens)
  Impact: no visibility into cost per feature; cannot optimize what you can't measure
  Fix: log every LLM call with tokens + cost + user_id + feature name

  [SCALE-12] temperature NOT SET
  grep_search: LLM calls without temperature parameter
  Impact: nondeterministic outputs for tasks that need determinism (extraction, classification)
  Fix: temperature=0 for factual/structured tasks; temperature=0.7+ for creative only
```

### S3 — Scan Output Format

```
=== AI CODE AUDIT REPORT ===
Scanned: [N] AI-adjacent files
Date: [date]

SECURITY (OWASP LLM Top 10 2025):
  🔴 CRITICAL: [N] issues  ← SHIP BLOCKED until resolved
  🟠 HIGH:     [N] issues  ← fix before scale
  🟢 MEDIUM:   [N] issues  ← clean up

SCALING BOTTLENECKS:
  🔴 CRITICAL: [N] issues
  🟠 HIGH:     [N] issues
  🟢 MEDIUM:   [N] issues

TOP PRIORITIES (fix in this order):
  1. [file:line] [LLM02] Hardcoded API key — [exact fix]
  2. [file:line] [LLM01] User input in prompt — [exact fix]
  3. [file:line] [SCALE-03] No retry on LLM call — [exact fix]
  ...

SHIP STATUS: BLOCKED ❌ / CLEAN ✅
(BLOCKED if any 🔴 issue exists)
```

---

## Mode 1: Intake & Problem Classification

### Stack Fingerprint (mandatory before recommending anything)
```
1. Read package.json / requirements.txt / go.mod
2. Identify: frontend framework, backend lang, DB, auth, hosting
3. Check for existing AI: grep_search for openai/anthropic/langchain imports
   → If found: run Mode S FIRST
4. Identify the AI Problem Class:
```

| Class | Pattern | Solution |
|---|---|---|
| Semantic Search | "find similar to X" | Embeddings + Vector DB |
| RAG | "answer from my docs" | Chunking + Vector DB + LLM |
| Extraction | "pull structured data from text" | LLM + JSON schema |
| Classification | "categorize into N buckets" | Fine-tuned model or few-shot |
| Generation | "write/summarize/translate" | LLM + prompt template |
| Agent | "take actions, use tools" | ReAct + tool calling |
| Multi-Agent | "coordinate AI roles" | CrewAI / LangGraph |

### Stack Selection
| Existing Stack | AI Approach |
|---|---|
| Next.js | Vercel AI SDK + useChat hook |
| Node/Express | LangChain.js or direct OpenAI SDK |
| Python/FastAPI | LangChain or direct SDK (native async) |
| Supabase | pgvector (built-in) + OpenAI embeddings |
| Firebase | Vertex AI / Gemini API |

**Model Selection (current as of 2026):**
| Task | Model | Cost |
|---|---|---|
| Chat/RAG (quality) | claude-sonnet-4-6 / gpt-4o | $$$ |
| High-volume, cheap | claude-haiku-4-5-20251001 / gpt-4o-mini | $ |
| Complex reasoning | claude-opus-4-7 | $$$$ |
| Embeddings | text-embedding-3-small | $0.02/1M |

---

## Mode 2: RAG Build

**Phases: R1 → R2 → R3 → R4 → R5 (in order)**

### R1 — Ingestion Pipeline
```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.node_parser import SentenceSplitter

documents = SimpleDirectoryReader(input_dir="./data", recursive=True).load_data()
splitter = SentenceSplitter(chunk_size=512, chunk_overlap=50)
nodes = splitter.get_nodes_from_documents(documents)
# chunk_size=512 tokens (NOT chars) — start here, tune up/down
```

### R2 — Chunking Decision (most impactful choice — 80% of RAG failures trace here)
| Data type | Strategy | Why |
|---|---|---|
| General text | Fixed 512 tokens, 50 overlap | Baseline, works for most |
| Technical docs | Hierarchical (parent section + child paragraphs) | Preserves context boundaries |
| Code | Split by function/class boundary | Never mid-function |
| Tables/CSV | Never chunk — store whole rows | Tables lose meaning when split |
| Multi-doc with references | Graph RAG | Entities + relationships preserved |

### R3 — Hybrid Retrieval (always beats pure vector search)
```python
# Dense (semantic) + Sparse (keyword) = Reciprocal Rank Fusion
from llama_index.retrievers.bm25 import BM25Retriever
from llama_index.core.retrievers import QueryFusionRetriever

hybrid_retriever = QueryFusionRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    similarity_top_k=5,
    num_queries=4,
    mode="reciprocal_rerank",
)
# Add reranker for production: CohereRerank(top_n=3)
```

### R4 — Generation with Citations (mandatory in production)
```python
# Always return source_nodes — users need to verify AI claims
response = query_engine.query("What is the refund policy?")
print(response.response)      # generated answer
print(response.source_nodes)  # exact chunks used → citations
```

### R5 — Vector DB Selection
| Situation | Choice |
|---|---|
| Already on Supabase | pgvector (zero extra cost, < 1M vectors) |
| Hosted, < 10M vectors | Pinecone (easiest, managed) |
| Open source, self-hosted | Qdrant (fastest filtering, Rust) |
| Local dev / prototype | ChromaDB (in-memory, zero config) |

### R6 — Advanced Patterns (use when baseline fails)
- **Corrective RAG:** if relevance score < 0.7 → fall back to web search (Tavily)
- **HyDE:** embed hypothetical answer instead of raw query → better recall
- **Graph RAG:** KnowledgeGraphIndex when documents reference each other

---

## Mode 3: Agent Build

### When to build vs not
```
BUILD when: task requires tools (search, DB, APIs), multi-step reasoning,
            longer than one context window
DO NOT BUILD when: a single well-crafted prompt solves it (90% of cases),
                   latency < 2s required, simple RAG answers the question
```

### ReAct Pattern (standard agent)
```python
from langchain.agents import AgentExecutor, create_tool_calling_agent

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    max_iterations=10,        # ALWAYS SET — prevents infinite loops
    handle_parsing_errors=True,
    verbose=False,            # False in production
)
```

### LangGraph (stateful, branching flows)
```python
# Use when: agent needs to branch, loop with state, or human-in-the-loop
from langgraph.graph import StateGraph, END
workflow = StateGraph(AgentState)
workflow.add_node("retrieve", retrieve_node)
workflow.add_node("grade", grade_node)        # quality gate
workflow.add_conditional_edges("grade", route_fn, {"generate": "generate", "search": "web_search"})
workflow.add_edge("generate", END)
app = workflow.compile()
```

### Multi-Agent (CrewAI) — use only when single agent insufficient
```python
from crewai import Agent, Task, Crew, Process
# Role-based: researcher → writer → reviewer
# Use Process.sequential for ordered tasks
# Use cheaper model (haiku/gpt-4o-mini) for non-reasoning agents
```

---

## Mode 4: Context Engineering

The #1 failure mode in production AI. Context = everything the model sees: prompts, docs, history, tool descriptions.

### Token Budget Rules
```
Total context = system prompt + chat history + retrieved docs + tool descriptions + response
Keep response budget ≥ 20% of context window at all times.

System prompt:      < 500 tokens (concise role + rules)
Chat history:       sliding window — last 10 messages max, then summarize
Retrieved docs:     top-3 chunks only (hybrid retrieval + reranker cuts noise)
Tool descriptions:  < 50 tokens per tool; max 10 tools per agent (overlap = agent confusion)
```

### Compression Techniques
```
Conversation history: summarize turns older than N → "User asked about X, I explained Y"
Large documents: chunk + retrieve, never full-doc in context
Format: prefer YAML over JSON in prompts (66% more token-efficient)
Structured input: use bullet lists + labels; models parse labeled structure more reliably
```

### Prompt Caching (Anthropic)
```python
# Cache static parts (system prompt, large docs) — saves cost on repeated calls
messages = [{"role": "user", "content": [
    {"type": "text", "text": large_doc, "cache_control": {"type": "ephemeral"}},
    {"type": "text", "text": user_question}
]}]
# Cache TTL: 5 minutes. Re-use for all queries against same doc.
```

---

## Mode 5: Prompt Engineering

### System Prompt Structure (production template)
```
You are [ROLE]. [ONE sentence on goal].

Rules:
- Answer STRICTLY using provided context. If context lacks the answer, say: "I cannot answer from the provided documents."
- Do NOT use outside knowledge.
- [domain-specific rule]

Format: [output format if structured]
```

### Technique Selection
| Task | Technique |
|---|---|
| Factual extraction, classification | temperature=0, structured output (JSON schema) |
| Reasoning, analysis | Chain-of-thought: "Think step by step before answering" |
| Consistent format | Few-shot: 2-3 input→output examples in system prompt |
| Ambiguous queries | Query decomposition: break into sub-questions first |

### Structured Output (enforce, never ask nicely)
```python
# BAD: "Please respond in JSON"  ← model will sometimes deviate
# GOOD: enforce via schema
from pydantic import BaseModel
class ExtractedData(BaseModel):
    name: str
    price: float
    in_stock: bool

response = client.beta.chat.completions.parse(
    model="gpt-4o", messages=[...], response_format=ExtractedData
)
result = response.choices[0].message.parsed  # typed, guaranteed valid
```

---

## Mode 6: Production Hardening

### Reliability Layer (mandatory on every LLM call)
```python
# Exponential backoff with jitter — never call LLM raw in production
import time, random
for attempt in range(5):
    try:
        return await llm_call()
    except RateLimitError as e:
        wait = float(getattr(e, 'retry_after', None) or min(1 * (2**attempt), 60))
        time.sleep(random.uniform(0, wait))  # full jitter prevents thundering herd
    except APIStatusError as e:
        if e.status_code not in {429, 500, 503, 504}: raise  # 400/401 = not retryable
        time.sleep(random.uniform(0, min(1 * (2**attempt), 60)))
```

### Model Router (60-70% cost reduction)
```python
# Route simple tasks to cheap model, complex to smart model
CHEAP  = "claude-haiku-4-5-20251001"   # $0.00025/1K input
SMART  = "claude-sonnet-4-6"            # $0.003/1K input

def route(query: str, ctx_len: int) -> str:
    if ctx_len > 4000: return SMART
    if any(w in query.lower() for w in ["summarize","translate","classify","extract"]): return CHEAP
    if any(w in query.lower() for w in ["analyze","debug","compare","reason"]): return SMART
    return CHEAP
```

### Streaming (required for all interactive responses)
```python
# Backend
async def generate():
    async for chunk in client.chat.completions.create(model=model, messages=msgs, stream=True):
        if chunk.choices[0].delta.content:
            yield f"data: {chunk.choices[0].delta.content}\n\n"
    yield "data: [DONE]\n\n"
return StreamingResponse(generate(), media_type="text/event-stream")

# Frontend (Next.js)
const { messages, handleSubmit } = useChat({ api: '/api/chat' })
```

### Fallback Provider
```python
providers = [("openai", call_openai), ("anthropic", call_anthropic)]
for name, fn in providers:
    try: return await fn(messages)
    except Exception: continue
raise Exception("All AI providers failed")
```

---

## Mode 7: Debugging Loop

**Trigger:** AI behaves wrong in production. Diagnose before changing anything.

### RAG Failure Diagnosis Table
| Symptom | Root cause | Fix |
|---|---|---|
| Answer ignores relevant docs | Retrieval thrash — wrong chunks retrieved | Switch to hybrid retrieval + reranker |
| Answer misses multi-doc reasoning | Single-doc chunking, no graph links | Graph RAG or parent-child hierarchy |
| Good answers degrade over time | Knowledge base stale or growing past optimal | Re-index; add timestamp filter |
| 80%+ wrong answers | Chunking strategy wrong | Audit chunk boundaries; try semantic chunking |
| Correct facts but hallucinated connections | No citation enforcement | Add citation gate to system prompt |
| Agent keeps re-querying same thing | Retrieval thrash — no convergence | Add retrieval memory; track already-retrieved IDs |

### Agent Failure Diagnosis Table
| Symptom | Root cause | Fix |
|---|---|---|
| Agent loops without terminating | No max_iterations | Set max_iterations=10 |
| 10-step task succeeds 20% | Compounding errors (0.85^10 = 0.20) | Reduce steps; add verification node |
| Tool calls storm (50+ per request) | Agent unclear on tool purpose | Reduce tool count; sharpen tool descriptions |
| Context bloat → degraded quality | Agent accumulates too much state | Compress state between nodes; summarize history |
| Agent ignores tool result | Tool output format mismatch | Standardize tool return as string; add parsing |

### Debug Protocol
```
1. Enable verbose logging: AgentExecutor(verbose=True) or LangSmith tracing
2. Isolate the failing node: reproduce with minimal input
3. Inspect: what was retrieved? what was the exact prompt sent? what tokens used?
4. Check tool calls: correct arguments? correct return format?
5. Fix ONE thing. Re-run. Measure.
Never fix multiple things simultaneously — you'll lose causality.
```

---

## Mode 8: Observability

**MELT framework — implement from day one, not after problems appear.**

```python
# Log every LLM call: model, tokens, cost, latency, user_id, feature
# ⚠️ Verify current pricing before trusting these values:
#    Anthropic: anthropic.com/pricing  |  OpenAI: openai.com/pricing
TOKEN_COSTS = {
    "claude-haiku-4-5-20251001":  {"input": 0.00000025, "output": 0.00000125},
    "claude-sonnet-4-6":          {"input": 0.000003,   "output": 0.000015},
    "claude-opus-4-7":            {"input": 0.000015,   "output": 0.000075},
    "gpt-4o":                     {"input": 0.0000025,  "output": 0.000010},
    "gpt-4o-mini":                {"input": 0.00000015, "output": 0.0000006},
}
# Emit per call: cost_usd, input_tokens, output_tokens, latency_ms, cached (bool)
```

**Tools:** LangSmith (traces + evals) · LangGraph Studio v2 (agent visualization + time-travel debug) · RAGAS (RAG quality scores)

---

## Mode C: Cost Reduction

**Trigger:** "reduce API costs" / "AI costs too high" / "optimize token usage" / "cut LLM bill" / "API bill is huge"

### C0 — Cost Audit (always run first)

```
1. grep_search for all LLM call sites (openai.chat.completions.create, anthropic.messages.create, client.invoke, etc.)
2. For each call site, check:
   □ Which model? (flag if Opus/GPT-4 used for non-reasoning task)
   □ max_tokens set? (flag if missing)
   □ stream=True? (flag if False on user-facing endpoint)
   □ Any cache check before this call? (flag if none)
   □ Part of batch-eligible flow? (async/offline job?)
   □ Does prompt include large static content repeated every call? (flag for prompt caching)
   □ Is this in a hot path that handles repetitive queries? (flag for semantic cache)
3. Print COST AUDIT MANIFEST with priority order before recommending anything.
```

### C1 — Stack Recommendations (infrastructure additions)

Based on audit findings, recommend only what's missing:

| If audit finds | Add to stack | Estimated savings |
|---|---|---|
| Repetitive queries (FAQ, search, chat) | **Redis** or **Upstash Redis** (semantic cache) | 40–73% of LLM calls eliminated |
| Large repeated static content in prompts (system prompt, docs) | **Anthropic prompt caching** or rely on **OpenAI auto-cache** | Up to 90% on cached tokens |
| Offline / async workloads (bulk analysis, content gen, eval) | **Batch API** (OpenAI or Anthropic) | 50% flat off all tokens |
| Premium model used for classify/summarize/extract | **Model Router** (Haiku / gpt-4o-mini) | 10–20x cost reduction |
| Long retrieved docs / conversation history in prompts | **LLMLingua-2** prompt compression | 20–80% token reduction |
| Serverless / Next.js edge runtime | **Upstash Redis** (not standard Redis — serverless-compatible) | Same as Redis |
| Multiple LLM providers needed | **OpenRouter** (automatic routing + fallback, unified API) | Cost varies by routing config |

**Decision rule:** add in this priority order — caching first (highest ROI, no quality loss) → model routing → batch → compression → fine-tuning.

---

### C2 — Layer 1: Caching (highest ROI, implement first)

Three-tier cache stack. Each tier catches what the previous misses.

**Tier 1 — Exact-match cache (free, instant)**
```python
# Check Redis for exact query string before ANY LLM call
import redis, hashlib, json

cache = redis.Redis.from_url(os.environ["REDIS_URL"])  # or Upstash

def cached_llm_call(query: str, context: str, ttl_seconds: int = 3600) -> str:
    key = f"llm:exact:{hashlib.sha256((query + context).encode()).hexdigest()}"
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    response = call_llm(query, context)
    cache.setex(key, ttl_seconds, json.dumps(response))
    return response
```

**Tier 2 — Semantic cache (catches paraphrases)**
```python
# "What's your return policy?" and "Can I return items?" → same cache hit
import numpy as np

SIMILARITY_THRESHOLD = 0.92

def semantic_cached_call(query: str, context: str) -> str:
    query_embedding = embed(query)  # text-embedding-3-small

    # Check all cached embeddings for cosine similarity
    for cached_key, cached_emb in get_cached_embeddings():  # from Redis HGETALL
        similarity = np.dot(query_embedding, cached_emb)
        if similarity > SIMILARITY_THRESHOLD:
            return cache.get(f"llm:resp:{cached_key}").decode()

    # Cache miss — call LLM, store embedding + response
    response = call_llm(query, context)
    key = hashlib.sha256(query.encode()).hexdigest()
    cache.hset("llm:embeddings", key, np.array(query_embedding).tobytes())
    cache.setex(f"llm:resp:{key}", 86400, response)
    return response
```

**Tier 3 — Prompt caching (reduces cost on cache HITS, not misses)**
```python
# Anthropic: 90% discount on cached tokens. Write cost: 1.25x standard (5-min TTL)
# Place static content FIRST — Anthropic caches from the top of the prompt down.

messages = [{
    "role": "user",
    "content": [
        # Static: cache this (system docs, large context, tool descriptions)
        {"type": "text", "text": large_static_document,
         "cache_control": {"type": "ephemeral"}},  # 5-min TTL
        # Dynamic: never cache this
        {"type": "text", "text": user_query}
    ]
}]

# OpenAI: automatic for prompts ≥ 1024 tokens. No code changes needed.
# Structure: static content at TOP, dynamic content at BOTTOM.
# Cache hits charged at ~50% of standard input price automatically.
```

**Savings summary:**
- Tier 1 (exact): ~0 cost for repeat queries — full elimination
- Tier 2 (semantic): 40–73% of LLM calls eliminated in chat/FAQ apps
- Tier 3 (prompt): 90% discount on static token portion per call

---

### C3 — Layer 2: Model Routing (10–20x reduction for mixed workloads)

```python
# Updated pricing (verify at anthropic.com/pricing + openai.com/pricing):
# claude-haiku-4-5:   $1.00 / $5.00  per MTok (input/output)
# claude-sonnet-4-6:  $3.00 / $15.00 per MTok
# claude-opus-4-7:    $15.00 / $75.00 per MTok  ← only for complex reasoning
# gpt-4o-mini:        $0.15 / $0.60  per MTok
# gpt-4o:             $2.50 / $10.00 per MTok

TASK_MODEL_MAP = {
    # cheap model — deterministic, low complexity
    "summarize":    "claude-haiku-4-5-20251001",
    "translate":    "claude-haiku-4-5-20251001",
    "classify":     "claude-haiku-4-5-20251001",
    "extract":      "claude-haiku-4-5-20251001",
    "format":       "gpt-4o-mini",
    # smart model — reasoning, code, analysis
    "analyze":      "claude-sonnet-4-6",
    "debug":        "claude-sonnet-4-6",
    "compare":      "claude-sonnet-4-6",
    "rag_answer":   "claude-sonnet-4-6",
    # premium — only when explicitly needed
    "complex_plan": "claude-opus-4-7",
}

def route_model(task_type: str, context_length: int) -> str:
    if context_length > 50_000:
        return "claude-sonnet-4-6"  # long context — sonnet handles 200k
    return TASK_MODEL_MAP.get(task_type, "claude-haiku-4-5-20251001")
```

---

### C4 — Layer 3: Batch API (50% off, async workloads only)

**Use when:** analysis pipelines, content generation, dataset processing, evals, scheduled jobs. **Never use for:** real-time chat, user-facing interactive responses.

```python
# OpenAI Batch API — 50% discount, 24hr completion window
import json
from openai import OpenAI

client = OpenAI()

# Step 1: Prepare JSONL batch file
requests = [
    {"custom_id": f"req-{i}", "method": "POST",
     "url": "/v1/chat/completions",
     "body": {"model": "gpt-4o-mini", "messages": [{"role": "user", "content": doc}],
              "max_tokens": 500}}
    for i, doc in enumerate(documents_to_process)
]
with open("batch_input.jsonl", "w") as f:
    for req in requests: f.write(json.dumps(req) + "\n")

# Step 2: Upload + create batch
batch_file = client.files.create(file=open("batch_input.jsonl", "rb"), purpose="batch")
batch = client.batches.create(
    input_file_id=batch_file.id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)

# Step 3: Poll until complete (or use webhook)
import time
while True:
    status = client.batches.retrieve(batch.id)
    if status.status == "completed": break
    time.sleep(60)  # poll every minute

# Step 4: Download results
results = client.files.content(status.output_file_id)
```

```python
# Anthropic Message Batches API — same concept, same 50% discount
import anthropic

client = anthropic.Anthropic()
batch = client.messages.batches.create(requests=[
    {"custom_id": f"doc-{i}",
     "params": {"model": "claude-haiku-4-5-20251001", "max_tokens": 500,
                "messages": [{"role": "user", "content": doc}]}}
    for i, doc in enumerate(documents_to_process)
])
# Poll: client.messages.batches.retrieve(batch.id)
# Results: client.messages.batches.results(batch.id)
```

---

### C5 — Layer 4: Prompt Compression (LLMLingua-2)

**Use when:** long retrieved docs / conversation history that can't be trimmed manually. **Skip when:** prompts are already under 1k tokens.

```bash
pip install llmlingua
```

```python
from llmlingua import PromptCompressor

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-xlm-roberta-large-meetingbank",
    use_llmlingua2=True,  # faster, better accuracy than v1
    device_map="cpu"      # or "cuda" if GPU available
)

# Light compression (2-3x): safest, <5% accuracy drop, ~80% cost reduction on compressed part
result = compressor.compress_prompt(
    long_retrieved_context,
    rate=0.5,          # keep 50% of tokens — tune based on task
    force_tokens=["\n", ".", "!", "?"],  # never compress sentence boundaries
    drop_consecutive=True
)
compressed_context = result["compressed_prompt"]
# Use compressed_context in your RAG prompt instead of raw retrieved docs
```

**Compression tradeoffs:**
| Rate | Keep % | Cost reduction | Accuracy retention |
|---|---|---|---|
| 0.5 | 50% | ~50% on that block | ~98% (safest) |
| 0.3 | 30% | ~70% on that block | ~90-95% |
| 0.1 | 10% | ~90% on that block | ~85% (aggressive) |

Start at rate=0.5. Only go lower if cost pressure requires it.

---

### C6 — Layer 5: Fine-tuning (last resort, high setup cost)

**Fine-tune ONLY when:**
- Single repetitive task at high volume (>100k calls/month)
- Current model is Sonnet/Opus but task only needs Haiku-level intelligence
- Consistent format required (extraction templates, classification)

**Do NOT fine-tune when:**
- Task variety is high (fine-tuned models specialize, they don't generalize)
- Volume is under 100k calls/month (setup cost won't pay off)
- You haven't tried prompt engineering + model routing first

**Decision:** if model routing to Haiku still misses quality bar → try few-shot prompting → if still failing → then consider fine-tuning. Fine-tuning a haiku-class model to do sonnet-quality work on your specific task = 10–20x cost reduction at scale.

---

### C7 — Cost Reduction Report Format

After running C0 audit + recommending layers, output this:

```
=== AI COST REDUCTION REPORT ===
Current estimated cost: [derive from token logs if available, else estimate]

QUICK WINS (implement this week):
  □ [technique] → estimated [X]% reduction → [specific files to change]
  □ ...

INFRASTRUCTURE TO ADD:
  □ Redis / Upstash Redis — [reason: semantic cache for X query patterns]
  □ [other addition] — [reason]

LAYERS RECOMMENDED (priority order):
  1. Caching (Tier 1+2+3) → [X]% reduction — zero quality impact
  2. Model routing → [X]% reduction — zero quality impact
  3. Batch API → 50% off [N] identified async flows
  4. Prompt compression → [X]% reduction — [compression rate recommended]
  5. Fine-tuning → [yes/no + reason]

ESTIMATED COMBINED SAVINGS: [X–Y]% (conservative–aggressive scenario)
```

---

### Cost Reduction Anti-Patterns

```
❌ Caching responses without TTL — stale answers served forever
❌ Semantic cache threshold too low (<0.85) — wrong answers returned as cache hits
❌ Batch API for real-time chat — 24hr latency, users get no response
❌ Aggressive LLMLingua compression (rate<0.3) without quality testing — accuracy degrades silently
❌ Fine-tuning before model routing — model routing alone may solve it for 1/100th the effort
❌ Opus/GPT-4 for classify/summarize — 10–20x overspend with no quality benefit
❌ Prompt cache without static-first ordering — cache never hits; wasted write cost
❌ No TTL on semantic cache — outdated product/policy info returned to users
❌ Compressing system prompts — never compress; only compress retrieved docs / history
```

---

## Pre-Ship Gate

**Triggered by:** "ready to ship", "pre-ship check", "deploy AI feature"

```
STEP 1 — Security (from Mode S scan):
  □ Zero 🔴 CRITICAL security issues
  □ No hardcoded API keys
  □ Rate limiting on all AI endpoints
  □ Input validation present
  □ No LLM output piped to unsafe sinks

STEP 2 — Reliability:
  □ Retry/backoff on all LLM calls
  □ max_iterations set on all agents
  □ max_tokens set on all LLM calls
  □ Streaming enabled for user-facing responses
  □ Fallback provider configured

STEP 3 — Eval (if RAG):
  □ faithfulness ≥ 0.80
  □ answer_relevancy ≥ 0.85
  □ context_recall ≥ 0.75
  Run: ragas.evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_recall])

STEP 4 — Cost (run Mode C audit if not already done):
  □ ModelRouter in place (not all-Opus/all-GPT4)
  □ max_tokens set on all LLM calls (no runaway responses)
  □ At minimum: exact-match cache before LLM calls
  □ Prompt caching enabled for static content ≥ 1024 tokens
  □ Batch API used for any async/offline processing flows
  □ Cost observability: token usage logged per call with user_id + feature

STEP 5 — Context:
  □ Chat history bounded (sliding window or summary)
  □ Tool count ≤ 10 per agent
  □ System prompt < 500 tokens
```

**Output:**
```
SHIP READY ✅  — all gates pass
SHIP BLOCKED ❌ — [list failing gates]
```

---

## Anti-Patterns

```
❌ API key in frontend — exposed in browser bundle
❌ Raw user input in prompt — prompt injection
❌ LLM output to innerHTML/eval/db.query — XSS / SQL injection / RCE
❌ No max_iterations on agents — infinite loop + runaway cost
❌ No rate limiting on AI routes — one user drains quota
❌ Full chat history every request — token cost explodes
❌ GPT-4/Opus for everything — 10-20x overspend
❌ No retry logic — first 429 crashes the request
❌ No streaming — 10-30s blank screen kills UX
❌ No eval before ship — "it seemed to work" is not production-ready
❌ Agents for simple tasks — a well-crafted prompt beats agents 90% of the time
❌ No fallback provider — one outage = total AI feature failure
❌ Chunking entire docs — context window exceeded + cost bombs
❌ No cost observability — you will overspend without knowing why
❌ Multiple overlapping tools — agent wastes cycles choosing
❌ Calling LLM API from browser — never, always route via backend
```
