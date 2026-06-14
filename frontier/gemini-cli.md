# Gemini CLI — Agent Orchestration Analysis

## Architecture Overview

Open-source (105K stars), TypeScript. Evolving from mono-agent to multi-agent architecture.

### Subagent System (shipped 2026)

- Each subagent has **own context window, system instructions, and curated tool set**
- Main agent acts as strategic orchestrator, delegating to relevant subagents
- Subagent execution consolidated into single response back to main agent
- Defined via **Markdown files with YAML frontmatter** (same pattern as Claude Code)
- Global (`~/.gemini/agents`) or project-level (`.gemini/agents`)
- Can be bundled in extensions via `agents/` directory
- Model routing: `model: inherit` or specific model override

**Key design:**
```yaml
---
name: frontend-specialist
description: Frontend specialist in building high-performance web apps
tools:
  - read_file
  - grep_search
  - glob
  - list_directory
  - web_fetch
  - google_web_search
model: inherit
---
```

### DAG Orchestration (in progress — Epic #22655)

Major architectural shift: **decoupling Plans (intent) from Trackers (execution state)**

Core pillars:
1. **Plan/Tracker Decoupling** — separate what needs to be done from its execution state
2. **Project-Level State** — lockable JSON files (`.gemini/`) for concurrent subagents
3. **Git Worktree Isolation** — physical isolation for parallel tasks
4. **DAG Orchestration** — parent-subagent loop managing dependencies and "Radius of Impact" for replanning
5. **Interactive TUI** for DAG visualization

### Non-Interactive Multi-Agent (PR #26345)

Sequential role-based execution: **planner → researcher → coder → tester → reviewer**
- Uses existing `runNonInteractive` path
- Agent capping to prevent runaway execution
- Dry-run support for safe analysis
- Conservative: no new permissions, deterministic and reviewable

### Background Agents (in progress — #18197)

- Remote agent calls can be pushed to background
- Reuses backgrounding from shell tool calls
- `ExecutionLifecycleService` for tool backgrounding
- `InjectionService` with source-aware injection
- CompletionBehavior for agnostic background task UI

### Other Features

- **Plan mode** — read-only planning
- **Checkpointing** — automatic session snapshots
- **Headless mode** — programmatic/scripting interface
- **Hooks** — customize behavior with scripts
- **MCP servers** — connect to remote agents
- **Model routing** — automatic fallback resilience
- **Extensions** — extend with new tools/capabilities

## What Works

1. **Markdown-based subagent definitions** — simple, inspectable, git-committable
2. **DAG-based orchestration** — formal dependency tracking, not ad-hoc
3. **Plan/Tracker separation** — intent vs state, enabling concurrent execution
4. **Non-interactive mode** — scripted multi-agent pipelines
5. **Open source** — full transparency, community contributions

## Failure Modes

- Still evolving — many features in progress or experimental
- Mono-agent architecture was the baseline until recently
- Community reports: multi-edit flows had bugs (#7039), fixed by sequentializing
- No dynamic workflow scripting like Claude Code yet

## Token Economics

- Subagents return consolidated responses — token-efficient
- Background agents don't block main session
- No explicit cost tracking mentioned

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Markdown agent definitions | Simple, inspectable, version-controllable |
| DAG orchestration | Formal dependency tracking for parallel execution |
| Plan/Tracker decoupling | Enable concurrent subagents with shared state |
| File-based state (JSON) | Lockable, concurrent-safe, inspectable |
| Open source (105K stars) | Community-driven, rapid iteration |

## API Surface for Pi Extensions

Key primitives:
- Markdown-based agent definitions (YAML frontmatter)
- DAG-based task dependency tracking
- Plan/Tracker separation
- File-based project state with locking
- Non-interactive multi-agent pipeline (planner → researcher → coder → tester → reviewer)
- Session checkpointing

## ⚠️ June 2026: Gemini CLI → Antigravity CLI Transition

**This is the biggest change in the space right now.**

On June 19, 2026, Google announced Gemini CLI is being replaced by **Antigravity CLI** (`agy`). This is NOT a rename — it's a full rewrite.

**Timeline:**
- Antigravity CLI available now
- Gemini CLI sunsets June 18, 2026 for AI Pro/Ultra users

**What changed:**
- **Built in Go** (not TypeScript) — faster, single binary, no Node.js
- **Asynchronous multi-agent orchestration** — orchestrator spawns specialized subagents in parallel, assembles outputs automatically
- **Unified architecture** — shares same agent harness as Antigravity 2.0 desktop
- **Dynamic subagents** from a single CLI prompt — no framework code needed
- Agent Skills, Hooks, Subagents, Extensions (now plugins) carried over

**Architecture (Antigravity):**
- Three-tier: Orchestrator → Parallel Subagents → Synthesis
- Orchestrator builds task graph from goal in plain English
- Each subagent gets fresh context window, runs in isolation
- Dependency graph allows parallel execution where possible
- Gemini 3.5 reasoning for synthesis

**v0.45.0 (Gemini CLI final, June 3):**
- Context Manager simplification
- A2A (Agent-to-Agent) usage metadata exposure
- Routing improvements (auto-routing, orphaned function error fixes)
- Terminal stability (Termux, PTY fixes)
- `experimental.adk.agentSessionSubagentEnabled` flag — routes subagent invocations through AgentSession protocol instead of legacy executors

**What this means for pi:**
- Google's architecture validates the orchestrator + parallel subagents + synthesis pattern
- Go binary with async orchestration is the production direction
- The transition itself is a cautionary tale about building on unstable platforms

## Sources

- https://developers.googleblog.com/en/subagents-have-arrived-in-gemini-cli/
- https://github.com/google-gemini/gemini-cli/issues/22655
- https://github.com/google-gemini/gemini-cli/pull/26345
- https://github.com/google-gemini/gemini-cli/issues/18197
- https://addyosmani.com/blog/gemini-cli/
- https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/
- https://github.com/google-gemini/gemini-cli/pull/27642
- https://contentbuffer.com/guides/antigravity-cli-subagents-parallel-tasks-from-terminal
- https://medium.com/google-cloud/building-a-multi-agent-gcp-infrastructure-auditor-with-google-antigravity-sdk-5792c3034fce
