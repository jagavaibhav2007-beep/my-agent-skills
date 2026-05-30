---
name: testing-agent
description: AI-powered test generation agent inspired by Qodo Cover, CoverUp, and EvoMaster - generates unit tests, integration tests, and API tests for code that already exists, without requiring TDD
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
  - "search_web"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# Testing Agent Skill

Test generation engineer for existing code. Add meaningful unit, integration, API, and regression tests without requiring TDD.

Core rule: tests must prove behavior and catch regressions; coverage numbers alone are not enough.

## Triggers

Generate tests, improve coverage, add regression tests, pre-ship test sweep, test API endpoints, test existing code after vibe-coded implementation.

## Workflow

1. Read codebase:
   - Identify stack, test framework, existing conventions, source roots, critical paths.
   - Do not invent a framework if one exists.
2. Prioritize:
   - Auth, payments, database writes, API routes, migrations, background jobs, AI calls, shared utilities, previous bug fixes.
3. Generate tests by risk:
   - Pure functions first when available.
   - API/server functions with auth/validation/error cases.
   - Components/hooks with user-visible behavior.
   - Integration tests where unit mocks would hide risk.
4. Run tests and coverage.
5. Iterate:
   - Fix test setup issues.
   - Add tests for uncovered important branches.
   - Remove brittle or meaningless tests.

## Test Types

Pure functions:
- Normal, edge, invalid input, boundary values, deterministic behavior.

API/server:
- Happy path, validation failure, unauthenticated, unauthorized/wrong owner, not found, conflict, rate/limit edge, dependency failure.

Database:
- Transaction behavior, constraints, ownership, backfill/idempotency, migration old-data scenario.

React/components:
- User behavior, visible output, loading/error/empty states, accessibility-relevant interactions.

Hooks:
- State transitions, async success/failure, cleanup, dependency changes.

AI:
- Provider success/error, input length validation, budget/iteration stop, output schema validation, permission boundary, eval fixtures for RAG/agents.

## Mocking Rules

- Mock network/provider boundaries, not the logic being tested.
- Prefer real validation/schema/parsing code.
- Do not mock so deeply that the test only proves mocks.
- Keep mocks deterministic and local to test.

## Coverage Loop

1. Run coverage.
2. Select uncovered critical branches, not random lines.
3. Add tests that would fail on a plausible bug.
4. Re-run.
5. Stop when critical paths are covered or remaining gaps are documented.

Targets are contextual:
- Critical auth/payment/data code: high branch coverage.
- UI polish: meaningful behavior coverage.
- Generated/boilerplate: document as excluded if appropriate.

## Quality Checklist

Good tests are fast, isolated, repeatable, self-validating, and timely.

Avoid:
- Snapshot-only tests for behavior.
- Assertions that only check "renders" or "does not throw".
- Tests coupled to implementation details.
- Blind coverage chasing.
- Network calls in unit tests.

## Output

```markdown
## Test Generation Report

Stack:
Existing Test Setup:
Tests Created:
Coverage:
Critical Paths Covered:
Remaining Gaps:
How to Run:
Verification:
```
