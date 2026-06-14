# Agent Research

Research on SOTA agent orchestration: subagents, dynamic workflows, distributed runtimes, and multi-agent patterns. Grounded in June 2026 source code and documentation.

## What's Here

### `frontier/` — Frontier Lab Implementations

| Tool | Key Insight |
|------|------------|
| **Claude Code** | Dynamic workflows GA (JS script-as-plan), ultracode, 1000 agent cap |
| **Codex CLI** | Multi-agent v2, SQLite-backed threads, /app handoff, Rust binary |
| **Gemini CLI → Antigravity** | June 18 sunset, Go rewrite, async multi-agent, AX runtime |
| **Cursor** | 3.7, Composer 2.5, auto-review classifier, 8 parallel agents in worktrees |
| **Windsurf** | Devin integration, Kanban Agent Command Center |
| **Synthesis** | Cross-cutting patterns, Anthropic harness patterns, context engineering |

### `papers/` — Academic Papers

| Paper | Key Insight |
|-------|------------|
| **RLM** | Context-as-environment, recursive sub-calls, 26% better than compaction |
| **DSPy** | Programs over prompts, automatic optimization, 35K stars |
| **Workflow-R1** | Multi-turn > one-shot, GSsPO at Think-Action cycle level |
| **Reasoning Topologies** | CoT→ToT→GoT→AGoT, dynamic structure selection at test time |
| **ADAS** | Agent design can be automated (ICLR Outstanding) |
| **Agent-as-a-Judge** | Agentic evaluation > LLM-as-a-Judge |

### `frameworks/` — Multi-Agent Frameworks

| Framework | Status | Key Pattern |
|-----------|--------|-------------|
| **LangGraph** | Production default | Graph-based state machine, checkpointing |
| **CrewAI** | Fast prototyping | Role-based agents, 60% Fortune 500 |
| **AutoGen** | Maintenance mode | Conversation-driven (deprecated) |
| **Google AX** | Early (v0.1.0) | Distributed Go runtime, event log resumption |
| **Vercel AI SDK 6** | GA | ToolLoopAgent, MCP first-class, 30M weekly npm |
| **Pydantic AI v2** | Beta | Capability model, type-safe agents |

### `ion/` — Source-Level Analysis

Deep dives into Pi, Codex, and Claude Code internals from [ion](https://github.com/nijaru/ion):

- **multi-agent-architecture** — Cross-tool comparison matrix (Pi vs Claude Code vs Codex vs Cursor vs Droid vs Amp)
- **pi-agent-loop-deep-dive** — 742-line agent-loop.ts analysis
- **codex-agent-loop-analysis** — Thread-based architecture, SQLite persistence
- **claude-code-tool-execution** — Partitioned execution, permission system
- **google-ax-runtime-review** — Distributed runtime patterns
- **claude-code-dynamic-workflows** — Thariq's six workflow patterns
- **rlm-dynamic-workflows** — RLM vs dynamic workflows comparison

## Key Findings

1. **Script-as-plan** is the winning pattern (Claude Code, RLM, Workflow-R1 all converge here)
2. **Git worktrees** are the multi-agent isolation primitive (everyone converged in Feb-Apr 2026)
3. **Never self-evaluate** — adversarial evaluators only (Anthropic confirmed)
4. **Hard cost budget** — nobody has it, everyone needs it
5. **Google AX** fills the runtime gap (event log, resumption, single-writer controller)
6. **Pi already has the best communication primitives** (intercom) — what's missing is the runtime layer

## Current As Of

June 14, 2026. Version numbers and release dates in each file.
