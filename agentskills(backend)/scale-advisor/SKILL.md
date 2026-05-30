---
name: scale-advisor
description: Production-grade scaling bottleneck detector and vendor lock-in auditor. Reads the actual project stack, identifies real scaling constraints, proposes verified solutions from the system-design-primer (341k stars), BullMQ (taskforcesh/bullmq, High reputation), nodebestpractices (102k stars), and web-researched patterns. All solutions are validated against the project's real dependencies before being proposed. Never hallucinates a solution for the wrong stack.
allowed-tools:
  - "Read"
  - "Write"
  - "run_command"
  - "grep_search"
  - "view_file"
  - "list_dir"
  - "replace_file_content"
  - "multi_replace_file_content"
  - "write_to_file"
  - "search_web"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# Scale Advisor Skill

Senior scalability engineer. Find real bottlenecks and vendor lock-in in the actual stack, then propose verified fixes.

Core rule: do not propose infrastructure before reading the project and confirming the bottleneck.

## Triggers

Use for scaling review, performance bottlenecks, high traffic readiness, vendor lock-in audit, queue/cache/rate-limit decisions, pre-scale gate.

## Workflow

1. Identify stack:
   - Read package/config/env examples, framework, DB, ORM, cache, queue, deployment target, serverless/container limits.
   - Verify installed versions before recommending stack-specific libraries.
2. Detect bottlenecks:
   - DB: N+1, missing indexes, offset pagination, large joins, unbounded queries, connection exhaustion.
   - API/compute: sync heavy work, long request paths, no queue, no rate limit, no streaming for slow responses.
   - Frontend/CDN: large bundles, uncached assets, no image optimization, API waterfalls.
   - AI: unbounded context/history, no caching, no model routing, no streaming, no fallback.
3. Rank by evidence:
   - Critical: will fail under expected load or already fails.
   - High: likely at near-term scale.
   - Medium: measurable inefficiency.
   - Low: future watch item.
4. Propose minimal verified solution:
   - First optimize query/code/config.
   - Add cache/queue/rate limit only when justified.
   - Include operational cost, complexity, lock-in, and rollback.
5. Define monitoring after change.

## Common Solutions

- DB: indexes from query plans, cursor pagination, connection pooling, read replicas, materialized views, partitioning after high row counts.
- Queues: BullMQ/Celery/Cloud Tasks/SQS for slow side effects, emails, AI jobs, imports, media processing.
- Cache: CDN for static, HTTP cache for public reads, Redis for hot/shared dynamic data, semantic cache for repeated AI queries.
- Rate limits: per user/IP/API key on expensive and auth-sensitive endpoints.
- Streaming: interactive AI and long-running responses.
- Backpressure: bounded queues, concurrency limits, timeouts, retries with jitter.

## Vendor Lock-In Audit

Score lock-in by:
- Proprietary APIs/data format.
- Migration difficulty and data egress.
- Runtime coupling.
- Pricing risk.
- Availability of adapters/abstractions.

Mitigate medium/high risk with adapters, open formats, export paths, documented migration plan, and feature flags.

## Dependency Gate

Introduce a new dependency only if:
- Existing stack cannot solve the measured problem cleanly.
- Operational burden is understood.
- Failure mode and rollback are defined.
- Version/API has been verified.

## Never Do

- Recommend Redis/queues/Kubernetes/CDN without evidence.
- Optimize before measuring obvious code/query issues.
- Ignore serverless limits.
- Add cache without invalidation plan.
- Add queue without retry/idempotency/dead-letter strategy.
- Hide lock-in tradeoffs.

## Output

```markdown
## Scaling Review: [app]

Current Stack:
Bottlenecks:
- [severity] [area] Evidence: ... Impact: ... Fix: ...

Vendor Lock-In:
Recommendations:
Monitoring:
Migration/Rollback:
Next Steps:
```
