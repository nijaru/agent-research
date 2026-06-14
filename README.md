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

## Key Findings

1. **Script-as-plan** is the winning pattern (Claude Code, RLM, Workflow-R1 all converge here)
2. **Git worktrees** are the multi-agent isolation primitive (everyone converged Feb-Apr 2026)
3. **Never self-evaluate** — adversarial evaluators only (Anthropic confirmed)
4. **Hard cost budget** — nobody has this, everyone needs it
5. **Classifier-gated autonomy** — Cursor auto-review, allowlist/sandbox/classify

## Current As Of

June 14, 2026. Version numbers and release dates in each file.
