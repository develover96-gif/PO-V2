-----

## name: prime-orchestrator
description: >
Strategic AI agent orchestration skill inspired by AgentHansa’s vision of “Building a New World for Agents.”
Use this skill whenever the task involves designing, analyzing, coordinating, or building multi-agent systems,
agent pipelines, LLM routing strategies, autonomous workflows, or agentic architectures. Trigger on phrases like
“orchestrate agents”, “multi-agent”, “agent pipeline”, “automate with AI agents”, “agent swarm”, “delegate to agents”,
“agent handoff”, “task decomposition”, “LLM routing”, “agentic workflow”, or whenever the user wants to build,
analyze, or scale an automated AI system across multiple models or services. This skill is model-agnostic and
framework-agnostic — it applies equally to Claude, GPT, Gemini, Mistral, open-source LLMs, and any agent runtime
(LangChain, CrewAI, AutoGen, LlamaIndex, custom implementations).
version: “1.0”
author: AgentHansa Strategic Intelligence Layer

# AgentHansa Orchestrator Skill

A universal, strategic orchestration framework for building robust, scalable, and adaptive AI agent systems.
This skill covers the full lifecycle: **analyze → architect → delegate → monitor → recover → scale**.

-----

## 0. Core Philosophy

> “An orchestra without a conductor produces noise. An agent network without orchestration produces chaos.”

AgentHansa’s orchestration model is built on five pillars:

|Pillar            |Principle                                                      |
|------------------|---------------------------------------------------------------|
|**Composability** |Agents are building blocks; orchestration is the blueprint     |
|**Specialization**|Each agent does one thing excellently                          |
|**Observability** |Every action is traceable, every decision is auditable         |
|**Resilience**    |Failures are expected; recovery is designed-in                 |
|**Adaptability**  |The system self-adjusts based on context, cost, and performance|

-----

## 1. Intake & Task Analysis

Before assigning any agent, perform structured task analysis:

### 1.1 Task Decomposition Protocol

Break every incoming task using the **SCOPE** framework:

- **S**tructure: Is this task sequential, parallel, or hierarchical?
- **C**omplexity: Single-step, multi-step, or open-ended with unknowns?
- **O**utput type: Text, code, data, action, decision, or artifact?
- **P**riority: Latency-sensitive, cost-sensitive, or quality-critical?
- **E**xternalities: Does this require tools, APIs, memory, or human-in-the-loop?

### 1.2 Task Classification Matrix

|Task Type                 |Recommended Pattern        |Primary Agent Role         |
|--------------------------|---------------------------|---------------------------|
|Linear transformation     |Sequential pipeline        |Transformer agent          |
|Research + synthesis      |Fan-out / fan-in           |Research + Synthesis agents|
|Decision under uncertainty|Hierarchical with critic   |Planner + Validator        |
|Continuous monitoring     |Event-driven loop          |Watcher + Alerter          |
|Creative generation       |Diverge-converge           |Generator + Refiner        |
|Code execution            |Tool-augmented single agent|Coder + Executor           |
|Multi-domain coordination |Swarm with mediator        |Specialist + Coordinator   |

-----

## 2. Agent Taxonomy

Define and classify all agents before orchestration:

### 2.1 Core Agent Roles

```
ORCHESTRATOR (meta-agent)
  ├── PLANNER        — Breaks goals into task graphs
  ├── ROUTER         — Selects the best agent/model for each task
  ├── CRITIC         — Evaluates output quality before passing downstream
  ├── MEMORY         — Manages shared state and context windows
  └── MONITOR        — Tracks performance, cost, and SLA compliance

EXECUTOR AGENTS (worker-level)
  ├── RESEARCHER     — Web search, document retrieval, RAG
  ├── ANALYST        — Data reasoning, summarization, interpretation
  ├── CODER          — Code generation, debugging, testing
  ├── COMMUNICATOR   — Email, Slack, reports, natural language output
  ├── TOOL-USER      — API calls, browser, file I/O, database queries
  └── SPECIALIST     — Domain-specific (legal, medical, financial, etc.)
```

### 2.2 Agent Capability Card (Required for Every Agent)

Each agent in the system MUST have a Capability Card:

```yaml
agent_id: "researcher-01"
role: RESEARCHER
model: "claude-sonnet-4-20250514"   # or gpt-4o, gemini-1.5-pro, etc.
tools: [web_search, document_retrieval]
input_schema:
  query: string
  depth: "shallow | deep"
  format: "structured | freeform"
output_schema:
  findings: string
  sources: list[url]
  confidence: float  # 0.0–1.0
latency_sla_ms: 8000
cost_per_call_usd: 0.004
fallback_agent: "researcher-02"
max_retries: 2
```

-----

## 3. Orchestration Patterns

Choose the appropriate pattern based on Task Classification (Section 1.2):

### 3.1 Sequential Pipeline

```
Input → [Agent A] → [Agent B] → [Agent C] → Output
```

**Use when:** Each step depends on the previous. Linear transformations, report generation, code-then-test.

**Rules:**

- Pass context (not just output) between agents
- Insert a CRITIC agent at decision boundaries
- Cap chain length at 6 agents before adding a summarizer

-----

### 3.2 Fan-Out / Fan-In (Parallel)

```
         ┌─→ [Agent A] ─┐
Input ──→├─→ [Agent B] ─┼──→ [Synthesizer] → Output
         └─→ [Agent C] ─┘
```

**Use when:** Subtasks are independent. Research, multi-perspective analysis, A/B generation.

**Rules:**

- All parallel branches must share a typed output schema
- Synthesizer receives ALL outputs simultaneously
- Set a `gather_timeout` — never wait indefinitely for a branch
- Mark slow branches as `optional: true` when latency > SLA

-----

### 3.3 Hierarchical (Planner + Workers)

```
[Orchestrator]
    ├── Task 1 → [Worker Agent A]
    ├── Task 2 → [Worker Agent B]
    │       └── Subtask 2.1 → [Sub-Agent X]
    └── Task 3 → [Worker Agent C]
```

**Use when:** Complex, multi-domain goals requiring dynamic sub-task generation.

**Rules:**

- Orchestrator must NOT do execution — only plan and route
- Workers report status and output back to Orchestrator
- Orchestrator maintains a `task_graph` with state: `pending | running | done | failed`
- Maximum nesting depth: 3 levels (Orchestrator → Worker → Sub-Agent)

-----

### 3.4 Critic Loop (Reflexive Quality Gate)

```
Input → [Generator] → [Critic] ──(fail)──→ [Generator] (retry)
                          │
                        (pass)
                          ↓
                       Output
```

**Use when:** Quality is critical. Code correctness, factual accuracy, policy compliance.

**Rules:**

- Critic must use a DIFFERENT model or temperature than Generator
- Define explicit pass/fail criteria before starting the loop
- Hard cap: 3 iterations max before escalating to human-in-the-loop
- Critic output must include: `{pass: bool, score: float, feedback: string}`

-----

### 3.5 Event-Driven Agent Loop

```
[Trigger Event] → [Watcher] → [Router] → [Specialist Agent]
      ↑                                          |
      └──────────────── [State Store] ←──────────┘
```

**Use when:** Continuous monitoring, reactive workflows, webhook-driven automation.

**Rules:**

- Watcher agent must be stateless and idempotent
- Always debounce events (avoid duplicate triggers)
- Use a dead-letter queue for unprocessable events
- State Store is the source of truth — agents read/write state, never hold it

-----

### 3.6 Swarm (Decentralized Multi-Agent)

```
[Agent A] ←→ [Agent B]
    ↕              ↕
[Agent C] ←→ [Agent D]
```

**Use when:** Emergent problem-solving, debate/consensus tasks, creative brainstorming.

**Rules:**

- Define a shared `goal_spec` that all agents can reference
- Use a moderator agent to prevent infinite loops
- Implement convergence criteria: `max_rounds`, `consensus_threshold`
- This pattern is expensive — only use when other patterns fail

-----

## 4. LLM Routing Strategy

Not all tasks need the same model. Route intelligently:

### 4.1 Model Selection Matrix

|Task Complexity           |Latency Need |Cost Priority      |Recommended Tier                             |
|--------------------------|-------------|-------------------|---------------------------------------------|
|Simple Q&A, classification|Low (<1s)    |High (cheap)       |Small model (Haiku, GPT-3.5, Mistral-7B)     |
|Reasoning, summarization  |Medium (<5s) |Medium             |Mid model (Sonnet, GPT-4o-mini, Gemini Flash)|
|Complex reasoning, code   |High (>5s OK)|Low (quality first)|Large model (Opus, GPT-4o, Gemini Pro)       |
|Vision + language         |Medium       |Medium             |Multimodal (Sonnet, GPT-4o, Gemini)          |
|On-device / private       |Any          |Highest            |Local (Llama 3, Phi-3, Mistral)              |

### 4.2 Dynamic Router Logic

```python
def route_task(task: Task) -> AgentConfig:
    if task.requires_tools and task.complexity == "high":
        return LARGE_MODEL_WITH_TOOLS
    elif task.latency_sla_ms < 1000:
        return SMALL_FAST_MODEL
    elif task.sensitive_data and task.requires_local:
        return LOCAL_MODEL
    elif task.output_type == "code" and task.requires_execution:
        return CODE_SPECIALIST_MODEL
    else:
        return DEFAULT_MID_MODEL
```

### 4.3 Fallback Chain

Always define a fallback chain per model tier:

```
Primary:  claude-sonnet-4-20250514
Fallback: gpt-4o-mini
Last:     local-llama-3-8b
Abort:    return cached_result OR escalate_to_human
```

-----

## 5. Context & Memory Management

Context is the lifeblood of multi-agent systems. Mismanage it and agents hallucinate, repeat work, or contradict each other.

### 5.1 Context Architecture

```
┌─────────────────────────────────────────┐
│          SHARED CONTEXT STORE           │
│  - Goal: string                         │
│  - Task graph: DAG                      │
│  - Completed outputs: dict              │
│  - Agent states: dict                   │
│  - Constraints: list                    │
│  - History summary: string (compressed) │
└─────────────────────────────────────────┘
          ↑ read/write          ↑ read-only
   [Worker Agents]        [Critic / Monitor]
```

### 5.2 Context Rules

1. **Never pass full history to every agent.** Compress with a MEMORY agent that maintains a rolling summary.
1. **Pass only what is needed.** Each agent receives a scoped context slice, not the full store.
1. **Always include the original goal** in every agent’s context — agents drift without it.
1. **Tag all outputs** with `agent_id`, `timestamp`, and `confidence_score`.
1. **Context window budget:** Reserve 20% of token budget for output; use the rest for context.

### 5.3 Memory Tiers

|Tier      |Type       |Scope                |Example                    |
|----------|-----------|---------------------|---------------------------|
|In-context|Ephemeral  |Current call only    |Recent messages            |
|Session   |Short-term |Current workflow run |Task outputs, agent states |
|Episodic  |Medium-term|User/project level   |Past decisions, preferences|
|Semantic  |Long-term  |Shared knowledge base|Domain facts, company data |

-----

## 6. Error Handling & Resilience

Design for failure. Every agent WILL fail. The orchestrator must anticipate this.

### 6.1 Error Taxonomy

|Error Type             |Cause                          |Response Strategy                                  |
|-----------------------|-------------------------------|---------------------------------------------------|
|`TimeoutError`         |Agent exceeded SLA             |Retry with smaller model → fallback                |
|`HallucinationDetected`|Agent produced unfactual output|Route to CRITIC → regenerate                       |
|`ToolCallFailed`       |External API/tool error        |Retry with backoff → skip tool → degrade gracefully|
|`ContextOverflow`      |Token limit exceeded           |Compress context → summarize history               |
|`AgentLoopDetected`    |Circular task delegation       |Break loop → escalate to human                     |
|`ConsensusFailure`     |Swarm agents disagree          |Increase rounds → bring in tie-breaker agent       |
|`CostBudgetExceeded`   |Token spend above threshold    |Switch to cheaper model → truncate task            |

### 6.2 Retry Policy

```yaml
retry_policy:
  max_retries: 3
  backoff_strategy: exponential   # 1s, 2s, 4s
  jitter: true
  on_exhaust: fallback_agent      # or: escalate_human | return_partial | abort
```

### 6.3 Circuit Breaker Pattern

Prevent cascading failures:

```
Agent call count threshold: 5 failures in 60s
→ OPEN circuit (stop calling agent)
→ Wait 30s
→ HALF-OPEN (allow 1 test call)
→ If success: CLOSE circuit (resume normal)
→ If failure: OPEN again
```

-----

## 7. Observability & Monitoring

You cannot improve what you cannot see.

### 7.1 Required Telemetry Per Agent Call

```json
{
  "trace_id": "abc123",
  "agent_id": "researcher-01",
  "task_id": "task-042",
  "model": "claude-sonnet-4-20250514",
  "input_tokens": 1240,
  "output_tokens": 380,
  "latency_ms": 3200,
  "cost_usd": 0.0032,
  "success": true,
  "confidence": 0.87,
  "tools_called": ["web_search"],
  "retry_count": 0,
  "timestamp": "2026-04-26T14:00:00Z"
}
```

### 7.2 Dashboard Metrics

Track these at the orchestration level:

- **Task completion rate** — % of tasks reaching final output without failure
- **End-to-end latency** — wall clock from task intake to output
- **Per-agent success rate** — detect degraded agents early
- **Cost per task** — track token spend across all agents in a workflow
- **Critic pass rate** — quality signal; low pass rate → generator prompt needs improvement
- **Retry rate** — high retries indicate unstable agents or bad routing
- **Human escalation rate** — the system’s “I give up” signal

### 7.3 Alerting Thresholds

|Metric               |Warning|Critical|
|---------------------|-------|--------|
|Task failure rate    |>5%    |>15%    |
|Agent retry rate     |>20%   |>50%    |
|P95 latency          |>10s   |>30s    |
|Cost per task        |>$0.10 |>$0.50  |
|Human escalation rate|>10%   |>25%    |

-----

## 8. Human-in-the-Loop (HITL) Integration

Not everything should be fully automated. Design intentional human checkpoints.

### 8.1 When to Insert HITL

- Output confidence < 0.7
- Task involves irreversible actions (send email, delete data, charge payment)
- Critic loop exhausted max retries
- Cost budget threshold exceeded
- Legal, medical, or financial domain outputs
- User explicitly flagged as requiring approval

### 8.2 HITL Interface Contract

```json
{
  "hitl_request_id": "hitl-789",
  "reason": "Low confidence on financial recommendation",
  "task_summary": "User requested investment allocation for $50,000",
  "agent_output": "...",
  "confidence": 0.61,
  "options": ["Approve", "Reject", "Edit"],
  "deadline_iso": "2026-04-26T15:00:00Z",
  "on_timeout": "reject_and_notify"
}
```

-----

## 9. Scaling Strategy

### 9.1 Horizontal Scaling Rules

- **Stateless agents** can be scaled infinitely via parallel instances
- **Stateful agents** (memory, coordinator) must use distributed state (Redis, vector DB)
- Use **worker pools** with queue-based dispatch for bursty workloads
- **Priority queues**: separate lanes for latency-critical vs. batch tasks

### 9.2 Cost Optimization Levers

1. **Prompt caching**: Cache system prompts and common prefixes
1. **Model downgrade on retries**: First try with mid model, downgrade on timeout
1. **Output compression**: Instruct agents to be concise; compress before passing
1. **Batch processing**: Group similar tasks and send in a single API call where supported
1. **Early exit**: If confidence > 0.95 after pass 1, skip Critic loop

### 9.3 Multi-Tenant Isolation

For orchestrating agents across users or clients:

- Each tenant gets a **scoped context store** (no cross-tenant memory bleed)
- Enforce **per-tenant cost budgets**
- Log all agent actions with `tenant_id` for compliance
- Rate-limit at the tenant level, not just the agent level

-----

## 10. AgentHansa Orchestration Blueprint (Quick Start)

When a user asks you to build or analyze an agent orchestration system, follow this sequence:

### Step 1 — Intake

```
Ask: What is the goal? What is the input? What does success look like?
Determine: Task type, complexity, output format, SLA requirements
```

### Step 2 — Architect

```
Select orchestration pattern (Section 3)
Define agent roster with Capability Cards (Section 2.2)
Choose LLM routing strategy (Section 4)
Design context flow (Section 5)
```

### Step 3 — Specify

```
Write the task graph (DAG of agents and dependencies)
Define schema contracts between agents (input/output types)
Set retry policies and fallback chains (Section 6)
Identify HITL checkpoints (Section 8)
```

### Step 4 — Implement

```
Build agents as stateless functions where possible
Use a shared context store for inter-agent communication
Instrument every agent call with telemetry (Section 7)
Add circuit breakers for external dependencies
```

### Step 5 — Validate

```
Run the Critic loop on all major outputs
Test failure modes: kill one agent, inject a bad output, exceed the budget
Verify observability: can you trace any output back to its source?
Load test with 10x expected volume before production
```

### Step 6 — Operate

```
Monitor dashboard metrics (Section 7.2)
Tune routing rules based on real performance data
Retire underperforming agents, promote reliable ones
Expand specialist agents as new domains emerge
```

-----

## 11. Anti-Patterns to Avoid

|Anti-Pattern               |Problem                                             |Fix                                               |
|---------------------------|----------------------------------------------------|--------------------------------------------------|
|**God Agent**              |One agent tries to do everything                    |Decompose into specialists                        |
|**Context Flooding**       |Passing full history to every agent                 |Use scoped context slices                         |
|**Infinite Retry**         |Retrying failed agents without escalation           |Hard cap retries; add circuit breaker             |
|**Silent Failure**         |Agent returns empty or wrong output without flagging|Enforce output schema validation                  |
|**Tight Coupling**         |Agents know each other’s internals                  |Use typed contracts; agents only know their schema|
|**No Fallback**            |Single-model dependency with no backup              |Always define a fallback chain                    |
|**Monolithic Orchestrator**|Orchestrator does both planning AND execution       |Strict separation of concerns                     |
|**Prompt Drift**           |Agent prompts modified without testing              |Version-control all prompts                       |
|**Missing HITL**           |Fully autonomous on high-stakes decisions           |Design HITL gates into the workflow               |

-----

## 12. Framework Compatibility Reference

This skill is designed to map directly onto major agent frameworks:

|Concept (this skill)|LangChain           |CrewAI       |AutoGen       |LlamaIndex          |Custom       |
|--------------------|--------------------|-------------|--------------|--------------------|-------------|
|Orchestrator        |AgentExecutor       |Crew         |GroupChat     |AgentRunner         |Custom loop  |
|Planner             |LLMChain            |Task planner |PlanAndExecute|QueryPlanningAgent  |Prompt-based |
|Worker Agent        |Tool-calling Agent  |Agent        |AssistantAgent|FunctionCallingAgent|LLM + tools  |
|Critic              |OutputParser + guard|QA Agent     |CriticAgent   |EvaluationAgent     |Separate call|
|Memory              |ConversationMemory  |Memory       |Buffer        |VectorStoreIndex    |Redis / DB   |
|Router              |RunnableBranch      |Conditional  |Custom        |RouterQueryEngine   |If/else      |
|Telemetry           |LangSmith           |Built-in logs|OpenTelemetry |Arize / Langfuse    |Custom logger|

-----

-----

## 13. Forum Intelligence Digest

The orchestrator continuously ingests signals from the global AI agent community to stay ahead of emerging patterns, failures, and breakthroughs. This is not passive monitoring — it is **active intelligence gathering** that feeds directly into strategy updates.

### 13.1 Signal Sources & Priority Tiers

|Tier                      |Source                                     |Signal Type                            |Crawl Frequency|
|--------------------------|-------------------------------------------|---------------------------------------|---------------|
|**T1 — High Signal**      |Hacker News (Ask HN, Show HN)              |Technical depth, early adoption        |Every 6 hours  |
|**T1 — High Signal**      |r/LocalLLaMA, r/MachineLearning            |Open-source agent patterns, benchmarks |Every 6 hours  |
|**T1 — High Signal**      |r/AIAgents, r/LangChain                    |Practitioner failures and wins         |Every 6 hours  |
|**T2 — Medium Signal**    |Twitter/X `#AgentOps` `#LLMOps`            |Rapid community reaction, new releases |Every 2 hours  |
|**T2 — Medium Signal**    |Discord: LangChain, AutoGen, CrewAI servers|Framework-specific bugs and workarounds|Every 12 hours |
|**T3 — Low Signal (Deep)**|arXiv cs.AI, cs.CL new submissions         |Research-grade patterns and benchmarks |Daily          |
|**T3 — Low Signal (Deep)**|GitHub trending (agent-related repos)      |Emerging tools, stars velocity         |Daily          |

### 13.2 Forum Digest Agent Pipeline

```
[Scheduler Trigger]
      ↓
[Crawler Agent] — fetches raw posts from all T1/T2 sources
      ↓
[Filter Agent] — removes off-topic, low-karma, and duplicate posts
      ↓
[Extractor Agent] — pulls: problem, solution, framework, model, outcome
      ↓
[Categorizer Agent] — maps each post to: pattern | failure | tool | benchmark | tip
      ↓
[Deduplication Agent] — merges near-identical signals across sources
      ↓
[Insight Synthesizer Agent] — produces structured Forum Intelligence Report
      ↓
[Strategy Delta Agent] — compares report to current SKILL strategy → generates diffs
      ↓
[Human Review Gate (optional)] — approve / reject proposed strategy changes
      ↓
[Strategy Updater Agent] — applies approved diffs to orchestration configs
```

### 13.3 Post Quality Scoring

Not all forum posts are worth ingesting. Score each post before processing:

```python
def score_post(post: ForumPost) -> float:
    score = 0.0

    # Engagement signals
    score += min(post.upvotes / 500, 1.0) * 0.30      # Karma (capped)
    score += min(post.comment_count / 50, 1.0) * 0.20  # Discussion depth

    # Content signals
    score += 0.25 if post.contains_code_snippet else 0.0      # Has impl
    score += 0.15 if post.contains_benchmark_result else 0.0  # Has data
    score += 0.10 if post.author_karma > 5000 else 0.0        # Expert poster

    # Recency decay
    days_old = (now() - post.created_at).days
    score *= max(0.2, 1.0 - (days_old / 14))  # Decay over 2 weeks

    return score  # 0.0–1.0; ingest if > 0.45
```

### 13.4 Structured Insight Extraction Schema

Each qualifying post is converted to a normalized `ForumInsight`:

```yaml
insight_id: "hn-2026-04-26-001"
source: "HackerNews"
source_url: "https://news.ycombinator.com/item?id=..."
score: 0.82
category: "failure"          # pattern | failure | tool | benchmark | tip
tags: ["multi-agent", "context-overflow", "claude", "langchain"]
problem: "Agents lose coherence after 8+ hops due to context dilution"
solution: "Insert a Summarizer agent every 4 hops; compress to <500 tokens"
framework_affected: ["LangChain", "AutoGen"]
models_mentioned: ["claude-sonnet", "gpt-4o"]
confidence: 0.79
extracted_at: "2026-04-26T06:00:00Z"
applied_to_strategy: false
```

### 13.5 Forum Intelligence Report (Weekly Output)

The Synthesizer Agent produces a structured weekly report:

```
=== AgentHansa Forum Intelligence Report — Week of 2026-04-21 ===

TOP PATTERNS THIS WEEK (by community signal):
  1. Corrective RAG loops outperforming static pipelines on factual tasks (+32% accuracy)
  2. Fan-out with 5+ parallel branches causing race conditions in stateful LangGraph
  3. Claude Sonnet 4 outperforming GPT-4o on tool-use chains with >3 sequential calls

TOP FAILURES THIS WEEK:
  1. Context overflow after hop 6+ in hierarchical agents (35 posts, 4 frameworks)
  2. Tool-call hallucinations when agent has access to >12 tools simultaneously
  3. Swarm agents deadlocking on consensus when all agents return equal confidence

EMERGING TOOLS (GitHub stars velocity > 500/week):
  - composio/agent-orchestrator: parallel coding agent fleet manager
  - BAND: universal cross-agent orchestration layer

RECOMMENDED STRATEGY DELTAS (pending review):
  DELTA-001: Cap agent tool access to ≤8 tools per agent (from unlimited)
  DELTA-002: Insert hop-count counter; trigger Summarizer at hop 4 (from hop 6)
  DELTA-003: Add tiebreaker agent to Swarm pattern when consensus_score < 0.60
```

-----

## 14. Self-Improvement Loop

The orchestrator does not remain static. It uses Forum Intelligence Digests and its own operational telemetry to **autonomously propose and apply improvements** to its own strategies, routing rules, agent configs, and prompts.

> “The best orchestrator is one that makes itself better every week without requiring a human to rewrite it.”

### 14.1 Self-Improvement Architecture

```
[Operational Telemetry] ──┐
                           ├──→ [Strategy Delta Agent] → [Improvement Proposals]
[Forum Intelligence]   ──┘                                       ↓
                                                    [Critic Agent validates proposals]
                                                                 ↓
                                              ┌─────────────────┴──────────────────┐
                                              │   Auto-apply (low-risk changes)     │
                                              │   Human gate (high-risk changes)    │
                                              └─────────────────┬──────────────────┘
                                                                ↓
                                                    [Versioned Strategy Store]
                                                                ↓
                                                    [A/B Test new vs old config]
                                                                ↓
                                                    [Promote winner / Rollback loser]
```

### 14.2 Improvement Categories & Risk Levels

|Category                  |Example Change                                |Risk Level|Auto-Apply?   |
|--------------------------|----------------------------------------------|----------|--------------|
|**Routing threshold**     |Adjust cost cutoff from $0.05 → $0.04         |Low       |✅ Yes         |
|**Retry policy**          |Reduce max retries from 3 → 2                 |Low       |✅ Yes         |
|**Tool access limit**     |Cap tools per agent from unlimited → 8        |Low       |✅ Yes         |
|**Hop counter**           |Insert Summarizer at hop 4 instead of 6       |Medium    |🔶 With logging|
|**Fallback chain**        |Add new model to fallback tier                |Medium    |🔶 With logging|
|**Pattern selection rule**|Prefer Fan-Out over Swarm for >4 domains      |Medium    |🔶 With logging|
|**Agent prompt update**   |Rewrite system prompt for Critic agent        |High      |❌ Human review|
|**New agent added**       |Add a Fact-Checker agent to pipeline          |High      |❌ Human review|
|**Architecture change**   |Switch Sequential → Hierarchical for task type|High      |❌ Human review|

### 14.3 Strategy Delta Agent Logic

```python
def generate_strategy_delta(
    telemetry: OperationalMetrics,
    forum_insights: list[ForumInsight],
    current_strategy: StrategyConfig
) -> list[StrategyDelta]:

    deltas = []

    # Telemetry-driven improvements
    if telemetry.critic_pass_rate < 0.70:
        deltas.append(StrategyDelta(
            type="prompt_update",
            target="generator_agent.system_prompt",
            reason=f"Critic pass rate {telemetry.critic_pass_rate:.0%} below threshold",
            risk="high",
            proposed_value="[revised prompt with stricter output constraints]"
        ))

    if telemetry.avg_hop_count > 5 and telemetry.task_failure_rate > 0.08:
        deltas.append(StrategyDelta(
            type="config_update",
            target="summarizer_trigger_hop",
            reason="High hop count correlates with elevated failure rate",
            risk="medium",
            proposed_value=4  # down from current
        ))

    # Forum-driven improvements
    for insight in forum_insights:
        if insight.category == "failure" and insight.confidence > 0.75:
            if "tool-call hallucination" in insight.problem:
                deltas.append(StrategyDelta(
                    type="config_update",
                    target="max_tools_per_agent",
                    reason=f"Community signal: {insight.source_url}",
                    risk="low",
                    proposed_value=8
                ))

    return deltas
```

### 14.4 A/B Testing Protocol for Strategy Changes

Before promoting any medium/high-risk change to production, run an A/B test:

```yaml
ab_test:
  name: "hop-summarizer-threshold-4-vs-6"
  hypothesis: "Summarizing at hop 4 reduces task failure rate vs hop 6"
  control:
    config: current_strategy
    traffic_split: 50%
  variant:
    config: proposed_strategy  # summarizer at hop 4
    traffic_split: 50%
  success_metric: task_failure_rate
  minimum_sample_size: 200 tasks
  minimum_duration: 48 hours
  significance_threshold: 0.95
  on_success: promote_variant
  on_failure: revert_to_control
  on_inconclusive: extend_test_by_48h_then_revert
```

### 14.5 Strategy Version Control

Every applied change is versioned and rollback-ready:

```
strategy-store/
  ├── v1.0.0/  ← initial release
  ├── v1.1.0/  ← forum delta: cap tools to 8
  ├── v1.1.1/  ← telemetry delta: retry backoff from 1s → 2s base
  ├── v1.2.0/  ← human-approved: add Fact-Checker agent
  └── v1.2.1/  ← forum delta: summarizer at hop 4
      └── CHANGELOG.md  ← reason, source, A/B result, applied_at
```

Rollback command: `orchestrator strategy rollback --to v1.1.1 --reason "degraded critic pass rate"`

### 14.6 Self-Improvement Guardrails

The system must never improve itself into harm:

1. **Identity lock**: Core routing safety rules (HITL gates, budget caps, data isolation) are **immutable** — no delta agent can modify them
1. **Delta review ceiling**: No more than 3 auto-applied changes per 24-hour window
1. **Regression detection**: If any key metric degrades >10% after an auto-applied change, auto-rollback within 15 minutes
1. **Audit trail**: Every change — auto or human — is logged with `source`, `justification`, `forum_insight_id` or `telemetry_snapshot_id`
1. **Drift alarm**: If strategy diverges >20% from baseline architecture patterns, freeze auto-apply and require human review
1. **Sandboxing**: All proposed changes run in an isolated shadow environment before any production traffic touches them

### 14.7 Self-Improvement Cadence

|Activity                  |Frequency     |Trigger                 |
|--------------------------|--------------|------------------------|
|Forum crawl & digest      |Every 6 hours |Scheduled               |
|Telemetry analysis        |Every 1 hour  |Scheduled               |
|Delta generation          |Every 12 hours|After digest + telemetry|
|Low-risk auto-apply       |As generated  |After Critic approval   |
|A/B test launch           |Weekly        |After human approval    |
|Strategy version promotion|Weekly        |After A/B test success  |
|Full strategy audit       |Monthly       |Scheduled + human review|
|Emergency rollback        |On demand     |Regression alarm        |

-----

*AgentHansa Orchestrator Skill — v1.1*
*“Building a New World for Agents — and Teaching Them to Improve Themselves”*
