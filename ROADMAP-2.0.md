# CodeLens 2.0 — Multi-Agent AI Code Review Platform

## Vision
Transform CodeLens from a codebase Q&A system into a production-grade multi-agent AI code review platform capable of reviewing Pull Requests, detecting issues, generating tests, and integrating directly into engineering workflows.

---

## Current State

**Implemented:**
- GitHub repository ingestion
- AST-based code chunking (Tree-sitter)
- Hybrid search (Vector + BM25)
- Repository Q&A
- Web search fallback
- LangGraph orchestration
- Source citations with file + line references
- Rate limiting + circuit breaker
- Qdrant vector storage

---

## Phase 1 — Pull Request Review Agent

**Goal:** Review GitHub PRs, not just answer questions.

**Input:**
```json
{ "repo": "org/repo", "pr_number": 123 }
```

**Output:**
```json
{
  "summary": "...",
  "security_issues": [],
  "performance_issues": [],
  "maintainability_issues": [],
  "test_recommendations": []
}
```

**New components:**

- **PR Fetcher** — changed files, commit history, diffs, reviewer comments
- **Diff Parser** — additions, deletions, modified methods, review context
- **Changed Code Index** — embed only modified files (faster, cheaper, focused)

---

## Phase 2 — Multi-Agent Review Workflow

**Goal:** Replace single-agent analysis with specialized reviewers running in parallel.

```
PR → Diff Analyzer
          |
     +---------+---------+
     |         |         |
Security  Performance  Maintainability
     |         |         |
     +---------+---------+
                |
           Test Agent
                |
           Aggregator
                |
          Review Report
```

**Agent 1 — Security Reviewer**
- SQL injection, XSS, command injection, secrets in code, auth/authz flaws
- Returns: severity, reasoning, remediation

**Agent 2 — Performance Reviewer**
- N+1 queries, inefficient loops, blocking operations, memory leaks
- Returns: bottleneck location, estimated production impact

**Agent 3 — Maintainability Reviewer**
- SOLID violations, long methods, dead/duplicate code, naming issues
- Returns: code quality score, refactor suggestions

**Agent 4 — Test Coverage Reviewer**
- Missing unit, edge case, and negative tests
- Returns: test gaps + generated test stubs

**Note:** Security, Performance, and Maintainability agents run in parallel via `asyncio.gather`. Test agent runs after since it depends on maintainability context.

---

## Phase 3 — Automated Test Generation

**Goal:** Generate production-ready test cases from reviewed code.

**Input:**
```python
def calculate_tax(amount: float) -> float:
    return amount * 0.18
```

**Output:**
```python
def test_calculate_tax_returns_18_percent():
    assert calculate_tax(100) == 18.0

def test_calculate_tax_zero():
    assert calculate_tax(0) == 0.0

def test_calculate_tax_negative():
    assert calculate_tax(-100) == -18.0
```

**Scope:** Unit tests, integration tests, mock generation, edge cases.

---

## Phase 4 — Static Analysis as Agent Context

**Goal:** Feed deterministic tool output into LLM agents as additional context — not as a standalone feature.

**Tools:**
- Python: Bandit, Ruff
- Universal: Semgrep
- .NET: Roslyn Analyzers

**Workflow:**
```
Code → Static Analysis (deterministic) ──→ Agent Context
     → LLM Analysis (reasoning)        ──→ Combined Review
```

**Why this matters:** Static analysis reduces hallucinations. LLM explains and prioritises what static tools flag. Neither alone is sufficient.

---

## Phase 4.5 — PR Grounding & Historical Context

**Goal:** Ground current reviews in team history to surface pattern-level insights.

**Store per merged PR:**
- PR number, files changed, diff summary
- Review outcome (accepted/rejected suggestions)
- Bugs filed within 30 days referencing the PR
- AST node fingerprints of changed functions/classes

**Index key:** `{file_path}::{class_name}::{function_name}` hashed as embedding input — retrieves structurally similar changes across unrelated PRs.

**Retrieval during review:**
```
Current diff (AST fingerprints)
        +
Similar historical PRs (vector search on AST nodes)
        +
Static analysis output
        +
Current diff text
        → Agent context
```

**Enables:**
- "A similar null-check was introduced in PR #142 and caused a production incident."
- "This pattern has been rejected by your team 3 times — here's the preferred approach."
- Risk scoring based on historically buggy file regions

---

## Phase 5 — GitHub App Integration

**Goal:** Fully automate reviews on PR creation.

```
Developer opens PR
       ↓
GitHub Webhook → CodeLens Review → GitHub Comment
```

**Features:**
- Auto-review on PR open + new commits
- Review summary as PR comment
- Inline comments on changed lines

---

## Phase 6 — Feedback Loop + Review Memory

**Goal:** Learn from team decisions to improve future reviews.

**Store per team:**
- Accepted suggestions
- Rejected suggestions
- Rejection reasons
- Team coding standards

**Connected to eval metrics (Phase 7):** acceptance rate over time measures whether the system is improving, not just functioning.

**Why this is critical:** Without this, CodeLens gives the same mediocre suggestions forever. With it, it becomes team-specific and self-improving.

---

## Phase 7 — Evaluation Framework

**Goal:** Measure review quality rigorously.

| Metric | Formula |
|---|---|
| Review precision | Correct findings / Total findings |
| Acceptance rate | Accepted suggestions / Total suggestions |
| Review latency | Time from PR open to review posted |
| Cost per review | Token cost + infrastructure cost |

**Tools:** DeepEval, RAGAS

**Acceptance rate over time is the primary signal** — if it's not improving, the feedback loop (Phase 6) isn't working.

---

## Deferred / Dropped

| Item | Decision |
|---|---|
| Architecture review agent | Deferred — hardest to do well, easiest to fake. Add after Phase 7. |
| Enterprise RBAC + dashboard | Dropped — adds no ML depth, looks padded |
| Standalone static analysis phase | Merged into Phase 4 as agent context |

---

## Recommended Tech Stack

| Layer | Tools |
|---|---|
| AI / Agents | LangGraph, Groq / Anthropic |
| Evaluation | DeepEval, RAGAS |
| Retrieval | Qdrant |
| Static Analysis | Semgrep, Bandit, Ruff, Roslyn |
| Backend | FastAPI, PostgreSQL, Redis |
| Deployment | Docker, Kubernetes |
| Monitoring | Prometheus, Grafana |

---

## Implementation Order

| Sprint | Focus |
|---|---|
| 1 | PR Fetcher, Diff Parser, Changed File Index |
| 2 | Security Agent + Performance Agent (parallel) |
| 3 | Maintainability Agent + Test Agent |
| 4 | Review Aggregator + Report generation |
| 5 | PR grounding — historical PR store + AST fingerprint index |
| 6 | GitHub App webhook integration |
| 7 | Static analysis as agent context (Phase 4) |
| 8 | Feedback loop + Review memory (Phase 6) |
| 9 | Evaluation framework (Phase 7) |

---

## Portfolio Outcome

**Project title:** CodeLens 2.0 — Multi-Agent AI Code Review Platform

**Key highlights:**
- Multi-agent architecture with parallel execution
- PR review automation end-to-end
- Security, performance, and maintainability analysis
- Automated test generation
- GitHub App integration
- Self-improving feedback loop
- Production evaluation framework
- Scalable deployment
