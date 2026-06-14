# SOTA Research: RLM-Style Planning and Dynamic Workflows

**Date:** 2026-05-29  
**Status:** Under Active Consideration for Ion's Roadmap  

This document details recent advances in state-of-the-art inference-time scaling for terminal coding agents, specifically focusing on **Recursive Language Models (RLMs)** from MIT CSAIL and **Dynamic Workflows (RLM-style plans)** officially integrated into Claude Code with the release of Claude Opus 4.8.

---

## 1. The RLM (Recursive Language Model) Paradigm

Introduced by researchers at MIT CSAIL (Alex L. Zhang, Tim Kraska, Omar Khattab), **Recursive Language Models (RLMs)** address a fundamental bottleneck in LLM reasoning: **Context Window Degradation ("Context Rot")**. 

Standard long-context models suffer from dramatic quality degradation when forced to process massive prompts (e.g., entire directories or codebases) directly in a single context window.

### Core Architecture

```
                       [ Massive Input Data ]
                                 │ (Stored as variable)
                                 ▼
                     ┌───────────────────────┐
                     │  Python REPL Sandbox  │
                     └───────────────────────┘
                                 ▲
                     ┌───────────┴───────────┐
                     │ Symbolic Exploration  │ ◄─── Peeks, Slices, Searches
                     │       (LLM as Coder)  │ ───► Programmatic Queries
                     └───────────┬───────────┘
                                 │
                     ┌───────────▼───────────┐
                     │  Recursive Execution  │ ───► Spawns sub-agents on
                     │       (Sub-agents)    │      filtered snippet scopes
                     └───────────────────────┘
```

1.  **Context as an Environment:** Instead of feeding raw context into the prompt, the long input is held externally as a variable in a programming environment (e.g., a Python REPL or isolated shell).
2.  **Active Symbolic Exploration:** The LLM acts as an active programmer. It writes code (e.g., regex searches, filters, partitioning) to programmatically interrogate the data.
3.  **Recursive Sub-Queries:** When the agent isolates a complex sub-portion of data, it programmatically invokes sub-agents (recursive calls) on that narrow, well-defined slice, avoiding context clutter at the parent level.
4.  **Systems-Level Memory Management:** Inspired by "out-of-core" computing, this framework scales inference to millions of tokens without requiring larger base context windows.

---

## 2. Claude Code Opus 4.8 "Dynamic Workflows"

Anthropic officially commercialized this recursive, code-orchestrated concept with the introduction of **Dynamic Workflows** in Claude Code running on Opus 4.8.

### How Dynamic Workflows Work
When presented with a large-scale, complex task (e.g., migrating an entire repository's test harness or rewriting a large module):

*   **Plan-in-Code:** Claude dynamically writes a plan as an **executable script** rather than trying to track steps in a standard chat context.
*   **Parallel Execution Loop:** The orchestrator script launches, fans out, and manages tens or hundreds of lightweight parallel sub-agents (similar to recursive functions).
*   **Out-of-Context State:** The plan's state, intermediate values, and subtask dependencies are stored in script variables. The main chat history only retains the high-level orchestration progress and the final synthesized result.
*   **Iterative Self-Verification:** The orchestrator script incorporates verification assertions. Sub-agents critique and check each other's work programmatically, iterating until convergence.

---

## 3. Designing RLM-Style Plans in Ion

Ion is uniquely positioned to implement this SOTA paradigm because we have successfully restructured Ion into a standalone module with clean, public foundation packages (`ion/session`, `ion/tool`, `ion/prompt`, `ion/workspace`). 

### Proposed Ion Orchestration Architecture

```
                    ┌─────────────────────────┐
                    │     Ion TUI / CLI       │
                    └────────────┬────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │      ion/workflow       │
                    │   (Plan Orchestrator)   │
                    └────────────┬────────────┘
                                 │ Writes & Executes
                                 ▼
                    ┌─────────────────────────┐
                    │    Go Sandbox Runner    │
                    └────────────┬────────────┘
         ┌───────────────────────┼───────────────────────┐
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Sub-Agent A   │     │   Sub-Agent B   │     │   Sub-Agent C   │
│ (ion/session A) │     │ (ion/session B) │     │ (ion/session C) │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

1.  **Durable Workflow Package (`ion/workflow`):** Establish a new public package in Ion to manage orchestrator workflows. The workflow creates an execution engine capable of running simple dynamic orchestration scripts.
2.  **Scriptable Orchestration:** Give the main Ion agent a specialized tool, `execute_workflow_script`, which executes a generated orchestration plan written in a simple Go-based or scripting-based dialect.
3.  **Parallel Child Session Spawning:** Leverage `ion/session` to spawn lightweight, isolated child sessions in parallel. Each child session owns its own narrow workspace overlay (`ion/workspace`) and operates on a specific subset of files, reporting results back as variables in the parent script.
4.  **Verification Loop:** Incorporate self-critique agents that assert codebase sanity (using `make check`, `go test ./...`) before merging child session branches back to the main branch.

---

## 4. Next Steps for Ion P1
While Dynamic Workflows represent an exciting Phase 2 (Pi+) SOTA target, the immediate next step is executing our reopened **Phase 1 Pi-Parity Core** goals to guarantee a bulletproof foundation. Our standalone root packages (`ion/session`, etc.) now allow us to safely build out highly complex, multi-agent orchestrations directly in Ion.
