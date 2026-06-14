# Multi-Agent Frameworks — June 2026 Analysis

## The Big Picture

Three frameworks dominate. One is dying. A fourth approach (vanilla SDK) is gaining ground.

| | LangGraph | CrewAI | AutoGen | Vanilla SDK |
|---|---|---|---|---|
| Status | Production default | Fast prototyping | Maintenance mode | Growing trend |
| Stars | 28K | 48K | 55K | N/A |
| Adoption | Enterprise (Klarna, Uber, JP Morgan) | 60% Fortune 500 | Declining | Octomind, others |
| Mental model | Graph (nodes + edges) | Roles + tasks | Conversation | Direct code |
| Setup time | 10-14 engineer-days | 2-3 engineer-days | 5-7 engineer-days | 1-2 days |
| Complex task success | 62% | 54% | ~50% | Varies |
| Checkpointing | First-class | Limited | Limited | DIY |
| Best for | Stateful production agents | Role-based prototypes | Research, Microsoft | Teams who tried frameworks |

## LangGraph

**What it is:** Agent orchestration as directed graph. Nodes are computation steps, edges are control flow. State is typed (Pydantic), checkpointed, and replayable.

**Why it won production:**
- v1.0 (Oct 2025) added native checkpointing, human-in-the-loop approval nodes
- v1.2 (May 2026) added graceful shutdown, improved Pydantic coercion
- LangSmith observability built-in
- 90M monthly downloads
- Production deployments: Klarna, Uber, JP Morgan, BlackRock, Cisco, LinkedIn

**Key features:**
- `StateGraph` with typed schemas persists across node boundaries
- Crash → resume from last checkpoint (not from scratch)
- `interrupt()` pauses at any node for human approval
- Sub-graphs for multi-agent decomposition
- MCP support mature

**When to use:** Complex stateful workflows that need durability, checkpointing, and audit trails.

**When NOT to use:** Simple linear tasks, rapid prototyping (too much boilerplate).

## CrewAI

**What it is:** Role-based multi-agent collaboration. Agents as employees with roles, goals, backstories, and tools.

**Why it's popular:**
- Intuitive mental model (manage a team)
- Fast setup (2-3 engineer-days)
- v0.105 (Mar 2026) added enterprise observability and scheduling
- 47.8K GitHub stars, 27M PyPI downloads
- 60% Fortune 500 adoption

**Key features:**
- Agents defined by role, goal, backstory
- Tasks assigned to agents
- Sequential and hierarchical execution
- Flows for orchestration

**When to use:** Linear business-process automation, rapid prototyping, role-based decomposition.

**When NOT to use:** Complex branching logic, need for checkpointing/durability, production critical systems.

## AutoGen

**What it is:** Conversation-driven multi-agent prototyping. Agents talk to each other.

**Current status:** Maintenance mode. Microsoft shifted to Microsoft Agent Framework. Community packages still work but no new projects should start here.

**Why it mattered:**
- Pioneered multi-agent conversation patterns
- 55K GitHub stars (largest community)
- Code execution, research workflows

**When to use:** Legacy projects already on AutoGen, research, Microsoft-stack teams.

**When NOT to use:** New projects in 2026.

## The Vanilla SDK Approach

**What it is:** Direct SDK calls (Anthropic, OpenAI) + small state machine. No framework.

**Why it's growing:**
- Octomind post (May 2024) is the most-cited agent-framework discussion
- Pattern: pick a shiny framework → hit wall at month 3 → rewrite to direct SDK → ship faster
- Three teams in the article author's experience went through this cycle in 2025-2026

**When to use:** When you want full control, minimal dependencies, and your team can build the state management themselves.

## The Real Decision: Framework vs Platform

The most important question in 2026 isn't which framework — it's **framework vs managed platform**.

- **Frameworks** (LangGraph, CrewAI): You build, maintain, debug the agent system
- **Managed platforms** (Progenix, Nexus, productized agents): They run the agents for you
- **Productized agents** (Decagon, Sierra, Open.cx): Purpose-built for specific domains (customer service, etc.)

For pi extensions, the answer is clear: **we're building a framework** (the extension system), not a platform. But the patterns from LangGraph's checkpointing and CrewAI's role-based model are directly applicable.

## What This Means for Pi

### Steal from LangGraph
- Typed state with checkpointing → workflow journaled resume
- `interrupt()` for human-in-the-loop → approval gates
- Sub-graphs → workflow composition

### Steal from CrewAI
- Role + goal + backstory → agent definition format
- Simple mental model → easy onboarding

### Skip
- AutoGen (maintenance mode)
- Heavy framework dependencies (LangChain, CrewAI packages)
- Conversation-as-execution (inefficient for code tasks)

### The Vanilla SDK Lesson
Pi extensions should feel like direct SDK calls with batteries included, not a framework you have to learn. The model writes code (JS), the runtime executes it. No new abstractions to learn.

## Sources

- https://www.birjob.com/blog/ai-agent-framework-showdown-2026
- https://www.open.cx/blog/ai-agent-frameworks-langgraph-crewai-autogen-2026
- https://aiagentrank.io/blog/langgraph-vs-crewai-vs-autogen-2026
- https://bigaiagent.tech/langgraph-vs-crewai-vs-autogen-2026-2/
- https://dev.to/cristian_iridon_286794874/langgraph-vs-crewai-vs-autogen-in-2026-pick-the-right-ai-agent-framework-or-skip-frameworks-4m2c

## Google AX (Agent Executor) — Distributed Agent Runtime

**Repo:** https://github.com/google/ax (1.6K stars, Go, Apache 2.0)
**Status:** v0.1.0 (May 20, 2026), active early development
**Created:** March 2026 by rakyll (Google, formerly Go team)

### What It Is

A **distributed agent runtime** — not a framework for building agents, but infrastructure for running them reliably. Coordinates agentic loops, manages executions with event logging, communicates with local and remote actors.

### Key Features

- **Distributed Runtime**: Controller, skills, tools, and agents execute in isolation
- **Resumption**: Automatic recovery from failures or interruptions (event log, snapshotting)
- **Single-Writer Architecture**: Single controller ensures consistent state management
- **Auditing & Policy**: All calls coordinated by common controller, easy to audit
- **Portable**: Runs anywhere, targets Kubernetes
- **Model/Framework Agnostic**: Works with LangChain, LangGraph, ADK, any A2A-protocol agent

### Architecture

- **Controller**: Single-writer, manages state consistency
- **Task Executor**: Structured, stateful execution model (replaced simple loop)
- **Event Log**: Durable execution state with automatic recovery (newline-delimited JSON)
- **Skills/Tools**: Native support, configurable via YAML
- **Agents**: Gemini agent built-in, BYOH (Bring Your Own Harness) on roadmap
- **A2A Protocol**: Agent-to-Agent for inter-agent communication
- **MCP**: Native MCP server support

### Roadmap (from README)

1. Antigravity as built-in harness
2. BYOH (Bring Your Own Harness)
3. Suspend/resume of subagents
4. Tool call approvals in subagents
5. Improvements to resumption protocols

### Relevance to Pi

AX is the **runtime layer** that pi extensions lack. Pi has the orchestration (workflows, subagents) but not the durable execution, event logging, or distributed coordination. AX's patterns:

- **Event log for journaled execution** — newline-delimited JSON event log is exactly what workflow resume needs
- **Single-writer controller** — prevents state corruption in concurrent agent scenarios
- **Task executor pattern** — structured, stateful execution with sub-task delegation
- **Resumption protocols** — crash recovery without losing work

**Connection:** Antigravity CLI (Gemini CLI successor) will use AX as its runtime. Confirms Go + distributed runtime direction.

## Vercel AI SDK 6 — TypeScript Agent Layer

**Version:** ai@6.0.193 (June 2026)
**Downloads:** 20M+ monthly, 30M+ weekly npm installs
**Status:** GA, production-grade

### What It Is

The foundational TypeScript layer for AI applications. Low-level primitives (generateText, streamText, useChat) that most TS agent code builds on.

### v6 Key Addition: ToolLoopAgent

- Replaces Experimental_Agent (v5) with production-grade agent loop
- Default 20 steps — agents loop without explicit config
- Define once, call anywhere — reusable across requests
- Structured output via Output types directly
- MCP support (full OAuth + HTTP transport)
- Human-in-the-loop pause mechanism
- Local DevTools debugger

### Relevance to Pi

Pi is TypeScript but has its own agent loop. Relevance is **patterns**:

- **ToolLoopAgent** — 20-step default loop, define-once-reuse-anywhere
- **Output types** — structured output via schema, not string parsing
- **MCP first-class** — tool interoperability standard
- **ai-sdk-helpers** — ResponseCache (40-60% cost reduction), TokenBudget, RateLimiter

## Pydantic AI v2 — Python Agent Framework

**Version:** v2.0.0b7 (June 10, 2026), 18K stars
**Status:** Beta, stable expected within weeks

### v2 Key Addition: Capabilities

A **capability** is a composable unit bundling agent's tools, lifecycle hooks, instructions, and model settings. Pass via `capabilities=[...]`. Enables extensions to reach every layer through one concept.

### Relevance to Pi

Pydantic AI's capability model maps to pi extensions. A pi extension = capability: tools + hooks + instructions + model settings in one composable unit. Cleaner than separate extension/skill/hook model.

## Mastra — TypeScript Batteries-Included

**What:** Agents, workflows, memory, evals, MCP, deployment. Built by Gatsby team.
**Rising:** 2026 as "batteries-included" alternative to Vercel AI SDK's primitives approach.

## OpenAI Agents SDK

**Package:** @openai/agents
**Key Pattern:** Handoffs — agent A hands off to agent B with context. Clean alternative to fire-and-forget.

## Source-Level Analysis

Deep dives into Pi, Codex, and Claude Code internals are in [ion](https://github.com/nijaru/ion)/ai/ — ion's own repo. Includes cross-tool comparison matrix, agent loop analysis, and design decisions.

## Sources

- https://www.birjob.com/blog/ai-agent-framework-showdown-2026
- https://www.open.cx/blog/ai-agent-frameworks-langgraph-crewai-autogen-2026
- https://aiagentrank.io/blog/langgraph-vs-crewai-vs-autogen-2026
- https://github.com/google/ax
- https://chatforest.com/builders-log/vercel-ai-sdk-6-builder-guide/
- https://github.com/pydantic/pydantic-ai/releases/tag/v2.0.0b1
- https://www.developersdigest.tech/blog/typescript-ai-agent-stack-2026
