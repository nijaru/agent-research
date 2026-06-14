# Multi-Agent Architecture for Ion

**Date:** 2026-06-11
**Status:** Research/Planning (post-core, post-Pi-parity)

## The Pattern You Described

User ↔ Main Agent ↔ Sub-agents with bidirectional communication

This is **hierarchical delegation with persistent bidirectional channels**. Here's how it maps to the landscape:

## Reference Implementations

### Pi (Primary Reference — You Already Have This)

**Architecture:** Parent owns orchestration, children get concrete tasks

- `spawn_agent` creates sub-agents with fresh or inherited context
- `intercom` enables bidirectional messaging between sessions
- Interrupt/resume for mid-flight steering
- Chain, parallel, async, forked-context workflows
- Parent synthesizes results; children don't spawn their own children (by default)

**Key insight:** Pi separates *execution* (subagents) from *communication* (intercom). Subagents report back; intercom enables any-to-any messaging.

### Claude Code (Closest to Your Idea)

**Two modes:**

| Mode | Context | Communication | Best For |
|------|---------|---------------|----------|
| **Subagents** | Own window, results return to caller | Report back to main only | Focused tasks, lower token cost |
| **Agent Teams** | Own window, fully independent | Teammates message each other directly | Complex work requiring collaboration |

**Agent Teams architecture:**
- Team lead creates team, spawns teammates, coordinates work
- Shared task list with self-coordination (file locking)
- Mailbox system for inter-agent messaging
- Hooks: `TeammateIdle`, `TaskCreated`, `TaskCompleted`
- Display modes: in-process (cycle with Shift+Down) or split panes (tmux/iTerm2)
- No nested teams (teammates can't spawn their own teams)
- Lead is fixed for team lifetime

**Key insight:** Claude Code separates *subagents* (fire-and-forget delegation) from *teams* (persistent collaborative agents). Teams are heavier but enable richer coordination.

### Codex (Structured Delegation)

**Architecture:** Explicit agent definitions + batch processing

- Custom agents defined in TOML files (`~/.codex/agents/` or `.codex/agents/`)
- Built-in agents: `default`, `worker`, `explorer`
- `max_threads` (default 6), `max_depth` (default 1)
- `spawn_agents_on_csv` for batch processing (one worker per CSV row)
- Approval requests surface from inactive threads
- Sandbox policies per agent (read-only, workspace-write)

**Key insight:** Codex treats agents as *configuration* (TOML files) not *code*. Easy to define, share, and version. The CSV batch pattern is unique and powerful for large-scale audits.

### Cursor (Planner-Worker-Judge)

**Architecture:** Hierarchical with continuous replanning

- **Planners** continuously explore codebase, create tasks, can spawn sub-planners
- **Workers** pick up tasks, focus entirely on completion
- **Judge** determines whether to continue after each cycle
- Shared file with optimistic concurrency control
- Fresh starts to combat drift and tunnel vision

**Key insight:** Cursor found that flat structures failed (agents became risk-averse, avoided hard tasks). Hierarchy with role separation solved coordination. The judge/refresh cycle prevents drift.

### Factory Droid (Orchestrator + Validation Contracts)

**Architecture:** Multi-day autonomous missions

- Orchestrator plans, decomposes, delegates to specialized workers
- Validation contracts *before* implementation (adversarial validation)
- Deferred Context Engine (progressive disclosure — load tools/schemas only when needed)
- Factory Router (dynamic model routing — 20-25% cost reduction)
- Longest mission: 16 days continuous

**Key insight:** Droid's Deferred Context Engine is SOTA context management. Don't load everything upfront; discover and load as needed. The validation contract pattern prevents wasted work.

### Amp/Sourcegraph (Context Engineering)

**Architecture:** Context-first with subagent orchestration

- Context engineering as core principle (gather, compact, delegate)
- Subagents: Librarians (code search), implementation agents
- Artifact-based memory
- Structured planning with human oversight

**Key insight:** Amp treats *context* as the primary resource. Subagents multiply context windows.

### Windsurf/Devin (Agent Command Center)

**Architecture:** IDE as agent hub

- Kanban-style Agent Command Center
- Cloud delegation to Devin (persistent VMs)
- ACP (Agent Client Protocol) for interoperability
- Spaces for context preservation across sessions

**Key insight:** Windsurf treats the IDE as a *control plane* for multiple agents. ACP enables plugging in any agent (Codex, Claude, OpenCode).

## Comparison Matrix

| Feature | Pi | Claude Code | Codex | Cursor | Droid | Amp |
|---------|-----|-------------|-------|--------|-------|-----|
| Subagent spawning | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Bidirectional messaging | ✅ intercom | ✅ teams mailbox | ❌ one-way | ❌ shared file | ❌ orchestrator-only | ❌ report-back |
| Persistent agents | ✅ sessions | ✅ teams | ❌ task-scoped | ❌ cycle-scoped | ✅ missions | ❌ task-scoped |
| Mid-flight steering | ✅ interrupt | ✅ direct message | ❌ | ❌ | ❌ | ❌ |
| Role-based agents | ✅ builtin | ✅ subagent defs | ✅ TOML defs | ✅ planner/worker | ✅ specialized | ✅ librarian/impl |
| Context isolation | ✅ fresh/fork | ✅ own window | ✅ own window | ✅ own window | ✅ deferred | ✅ own window |
| Nested spawning | ✅ depth-limited | ✅ depth 5 | ✅ depth 1 default | ✅ sub-planners | ✅ orchestrator | ❌ |
| Quality gates | ✅ hooks | ✅ hooks | ✅ approval | ✅ judge | ✅ validation | ✅ human oversight |
| Batch processing | ❌ | ❌ | ✅ CSV fan-out | ❌ | ❌ | ❌ |
| Model routing | ✅ per-agent | ✅ per-teammate | ✅ per-agent | ✅ per-role | ✅ factory router | ❌ |

## What's Useful for Ion

### Phase 1: Subagent Foundation (After Core + Pi Parity)

**What:** Basic subagent spawning with fresh context

- Define agent types (worker, reviewer, researcher)
- Spawn sub-agents with task + context
- Collect results (fire-and-forget)
- No bidirectional messaging yet

**Reference:** Pi's `spawn_agent`, Codex's TOML agents

### Phase 2: Bidirectional Communication

**What:** Persistent channels between main agent and sub-agents

- Intercom-style messaging (async send, sync ask/reply)
- Parent can steer children mid-flight
- Children can escalate to parent
- Interrupt/resume support

**Reference:** Pi's `intercom`, Claude Code's team mailbox

### Phase 3: Agent Teams

**What:** Multiple agents collaborating with each other

- Shared task list with self-coordination
- Teammates can message each other (not just parent)
- Quality gates (hooks for task creation/completion)
- Display modes (in-process, split panes)

**Reference:** Claude Code's Agent Teams, Cursor's planner-worker-judge

### Phase 4: Advanced Patterns

**What:** SOTA features for complex workflows

- Deferred context engine (progressive disclosure)
- Dynamic model routing (cost optimization)
- Validation contracts (adversarial validation before implementation)
- Batch processing (CSV fan-out for large audits)
- Multi-day autonomous missions

**Reference:** Factory Droid, Cursor's long-running agents

## Design Decisions for Ion

### 1. Channel Model

**Recommendation:** Intercom-style async messaging (like Pi)

- `send` (fire-and-forget)
- `ask/reply` (synchronous)
- `broadcast` (to all agents)

**Why:** Most flexible. Enables both fire-and-forget delegation and interactive collaboration.

### 2. Agent Lifetime

**Recommendation:** Three modes

- **Task-scoped:** Spawn, execute, collect result, destroy (like Codex)
- **Session-scoped:** Persistent until explicitly destroyed (like Claude Code teams)
- **Mission-scoped:** Long-running with checkpoint/resume (like Droid)

**Why:** Different tasks need different lifetimes. Don't force one pattern.

### 3. Context Isolation

**Recommendation:** Fresh by default, fork on demand

- Fresh context: agent gets only the task description + project context
- Fork: agent inherits parent's context (expensive but sometimes necessary)

**Reference:** Pi's `context: "fresh" | "fork"` pattern

### 4. Bounded Depth

**Recommendation:** Default depth 3, configurable up to 5

- Main agent can spawn sub-agents
- Sub-agents can spawn their own sub-agents (bounded)
- Configurable via `max_depth` (default 3, hard cap 5)
- Each level tracks depth in agent metadata
- Orphan fallback: if parent dies, children notify grandparent or self-terminate

**Why:** Claude Code supports depth 5. Cursor uses sub-planners. Depth 3 is a good default — deep enough for complex decomposition, shallow enough to prevent runaway recursion. OpenClaw found depth 5 with Zod validation works well.

### 5. Quality Gates

**Recommendation:** Hooks at key lifecycle points

- `BeforeSpawn`: validate agent config
- `AfterTask`: validate result before accepting
- `OnIdle`: decide whether to continue or stop
- `OnEscalate`: handle child-to-parent communication

**Reference:** Claude Code's hooks, Droid's validation contracts

### 6. Model Routing

**Recommendation:** Per-agent model selection with optional auto-routing

Three modes:

1. **Explicit:** User specifies model per agent/task in config
2. **Role-based:** Model selected by agent role (planner=opus, worker=sonnet, reviewer=haiku)
3. **Auto:** Router selects model based on task complexity, context size, budget

**Config example:**
```toml
[agents.reviewer]
model = "sonnet"  # explicit

[agents.planner]
model = "by-role"  # role-based: planner gets opus, worker gets sonnet

[agents.worker]
model = "auto"     # auto: router decides based on task

[agents.router]
strategy = "cost-aware"  # minimize cost while maintaining quality
budget_per_task = 0.50     # USD cap per task
fallback = "sonnet"       # fallback model if router fails
```

**Factory Router insights:**
- 20-25% cost reduction without quality loss
- Route simple tasks to smaller models, complex to frontier
- Provider failover for reliability
- Session-aware routing for consistency

**Implementation:**
- Agent config specifies model (explicit, role-based, or auto)
- Router is a middleware that intercepts model selection
- Fallback chain: requested → auto-selected → default
- Log routing decisions for transparency

**Reference:** Factory Router, SAAR (Session-Aware Agentic Routing), pi-model-router

### 7. Any-to-Any Communication

**Recommendation:** Intercom-based messaging with agent discovery

**Topology options:**

| Topology | Description | Use Case |
|----------|-------------|----------|
| **Hub-and-spoke** | All messages go through parent | Simple delegation, fire-and-forget |
| **Peer-to-peer** | Any agent can message any agent | Complex collaboration, team patterns |
| **Mesh** | Agents discover each other by capability | Dynamic workflows, self-organizing teams |

**Recommendation:** Start with hub-and-spoke (Pi's current model), evolve to peer-to-peer

**Hub-and-spoke (Phase 1):**
```
User ↔ Main Agent ↔ Sub-agent A
                  ↔ Sub-agent B
                  ↔ Sub-agent C
```
- Parent routes all messages
- Children can only message parent
- Simple, predictable, easy to debug

**Peer-to-peer (Phase 2):**
```
User ↔ Main Agent ↔ Sub-agent A ↔ Sub-agent B
                  ↔ Sub-agent C ↔ Sub-agent A
```
- Agents can discover each other by name or capability
- Direct messaging without parent routing
- Parent still orchestrates but doesn't mediate every message
- Enables team patterns (Claude Code Agent Teams)

**Agent discovery:**
```go
type AgentRegistry interface {
    // Register an agent with capabilities
    Register(id AgentID, caps AgentCapabilities)
    // Find agents by capability
    FindByCapability(cap string) []AgentID
    // Find agents by role
    FindByRole(role string) []AgentID
    // List all active agents
    List() []AgentInfo
}
```

**Reference:** Pi's intercom, Claude Code's team mailbox, agent-mesh frameworks

## Implications for Ion Architecture

### Current State

```
Session Controller (wires everything)
├── AgentLoop (pure turn sequencing)
├── AgentRecovery (overflow/retry wrapper)
├── PersistenceListener (event-driven persistence)
├── Queues (steering/follow-up)
└── Event Stream (channel-based)
```

### Future State (Post-Core)

```
Session Controller
├── AgentLoop (pure turn sequencing)
├── AgentRecovery (overflow/retry wrapper)
├── PersistenceListener (event-driven persistence)
├── Queues (steering/follow-up)
├── Event Stream (channel-based)
├── AgentManager (NEW — sub-agent lifecycle)
│   ├── AgentRegistry (agent type definitions)
│   ├── AgentSpawner (create/destroy agents)
│   ├── Intercom (bidirectional messaging)
│   └── TaskBoard (shared task list)
└── ContextEngine (NEW — deferred context loading)
```

### Key Interfaces

```go
// AgentManager manages sub-agent lifecycle
// Supports recursive spawning up to max_depth
type AgentManager interface {
	Spawn(config AgentConfig) (AgentID, error)
	SpawnWithModel(config AgentConfig, model string) (AgentID, error)
	Destroy(id AgentID) error
	Interrupt(id AgentID) error
	Resume(id AgentID) error
	List() []AgentInfo
	GetTree() AgentTree // parent-child relationships
}

// Intercom enables bidirectional messaging
// Phase 1: hub-and-spoke (parent routes all)
// Phase 2: peer-to-peer (any agent can message any agent)
type Intercom interface {
	Send(to AgentID, msg Message) error
	Ask(to AgentID, msg Message) (Message, error)
	Reply(to AgentID, msg Message) error
	Broadcast(msg Message) error
	Receive() <-chan Message
	// Phase 2: discovery
	FindByCapability(cap string) []AgentID
	FindByRole(role string) []AgentID
}

// AgentConfig defines an agent type
type AgentConfig struct {
	Name        string
	Model       string          // explicit model, "by-role", or "auto"
	ModelTier   string          // "frontier", "standard", "fast" (for role-based)
	Tools       []string
	Context     ContextMode     // fresh | fork
	MaxDepth    int             // recursive spawn depth (default 3)
	QualityGate []Hook
	Budget      *BudgetConfig   // optional cost cap per task
}

// ModelRouter selects the best model for each task
type ModelRouter interface {
	Route(ctx RouteContext) (ModelID, error)
	SetStrategy(strategy RouteStrategy)
	LogDecision(ctx RouteContext, selected ModelID, reason string)
}

type RouteContext struct {
	Task        string
	AgentRole   string
	ContextSize int
	Budget      *BudgetConfig
	History     []RouteDecision
}

type RouteStrategy string

const (
	RouteExplicit  RouteStrategy = "explicit"   // use specified model
	RouteByRole    RouteStrategy = "by-role"    // model from agent role
	RouteCostAware RouteStrategy = "cost-aware" // minimize cost
	RouteBalanced  RouteStrategy = "balanced"   // balance cost and quality
	RouteQuality   RouteStrategy = "quality"    // maximize quality
)
```

### Recursive Spawning Architecture

```
Main Agent (depth 0)
├── Sub-agent A (depth 1)
│   ├── Sub-agent A1 (depth 2)
│   │   └── Sub-agent A1a (depth 3) ← max depth
│   └── Sub-agent A2 (depth 2)
└── Sub-agent B (depth 1)
    └── Sub-agent B1 (depth 2)
```

**Rules:**
- Each agent tracks its depth in metadata
- Spawn fails if depth >= max_depth
- Orphan fallback: if parent dies, children notify grandparent or self-terminate
- Token budget propagates down the tree (parent allocates budget to children)
- Routing decisions are logged at each level for transparency

## What to Build First

After core agent + Pi parity:

1. **AgentManager** — basic spawning with task-scoped lifetime, depth tracking
2. **AgentConfig** — TOML-based agent definitions (like Codex)
3. **ModelRouter** — explicit and role-based model selection
4. **Intercom** — async send/receive between parent and child
5. **Quality hooks** — BeforeSpawn, AfterTask, OnIdle

This gives you Pi-level subagent support with model routing. Then iterate toward teams, peer-to-peer messaging, and advanced patterns.

## Open Questions

1. ~~**Should Ion support nested spawning?**~~ Yes. Claude Code supports depth 5, Cursor uses sub-planners. Default depth 3, configurable up to 5.

2. **Should agents share a task list?** Claude Code teams use shared task list with file locking. Cursor uses optimistic concurrency. Recommend: start without shared task list, add when team patterns emerge.

3. **Should Ion support ACP?** Windsurf's Agent Client Protocol enables plugging in any agent. This would let Ion orchestrate Codex, Claude, etc. Recommend: defer until Ion's own agent system is solid.

4. **How should context be managed?** Droid's Deferred Context Engine is SOTA. Recommend: implement progressive context loading as part of the context engine.

5. **Should Ion support batch processing?** Codex's CSV fan-out is powerful for large audits. Recommend: add after basic subagent support is working.

6. **How should model routing work?** Three modes: explicit (user specifies), role-based (model from agent role), auto (router decides). Start with explicit + role-based, add auto later.

7. **How should any-to-any communication work?** Start with hub-and-spoke (parent routes all), evolve to peer-to-peer (any agent can message any agent). Agent discovery by name, role, or capability.

## User's Specific Requirements

### 1. Per-Task Model Selection

Users should be able to set models for the agent to use for different tasks:

```toml
# ~/.ion/config.toml

[models]
default = "sonnet"
planner = "opus"
worker = "sonnet"
reviewer = "haiku"
researcher = "opus"

[models.auto]
strategy = "cost-aware"
budget_per_task = 0.50
fallback = "sonnet"
```

Or per-task in the prompt:
```
Use opus to plan the architecture, then spawn workers with sonnet to implement each module.
```

### 2. Subagents Spawn Subagents

Subagents should be able to spawn their own subagents:

```
Main Agent (depth 0)
├── Planner (depth 1, model: opus)
│   ├── Worker A (depth 2, model: sonnet)
│   │   └── Reviewer (depth 3, model: haiku)
│   └── Worker B (depth 2, model: sonnet)
└── Researcher (depth 1, model: opus)
```

**Rules:**
- Default max depth: 3
- Configurable up to 5
- Each level can choose its own model
- Token budget propagates down
- Orphan fallback: if parent dies, children notify grandparent

### 3. Any-to-Any Communication

All agents/sessions should be able to communicate with each other:

```
User ↔ Main Agent ↔ Planner ↔ Worker A ↔ Reviewer
                  ↔ Worker B ↔ Worker A
                  ↔ Researcher ↔ Planner
```

**Communication patterns:**
- **Direct:** Agent A sends message to Agent B by name
- **Broadcast:** Agent A sends message to all agents
- **Capability-based:** Agent A sends message to agents with capability "review"
- **Role-based:** Agent A sends message to all agents with role "worker"

### 4. Complexity Reduction

This will have a lot of complexity we will have to solve and ideally reduce:

**Sources of complexity:**
1. Recursive spawning (depth tracking, orphan handling)
2. Model routing (per-task, per-role, auto)
3. Any-to-any communication (discovery, routing)
4. Token budget management (propagation, enforcement)
5. Context isolation (fresh vs fork, context windows)

**Reduction strategies:**
1. **Single abstraction:** AgentConfig + AgentManager + Intercom
2. **Sensible defaults:** depth 3, hub-and-spoke, explicit model
3. **Progressive disclosure:** start simple, add complexity as needed
4. **Configuration over code:** TOML files, not Go code
5. **Convention over configuration:** role-based model selection, default depth

## SOTA Comparison

| Feature | Ion (Target) | Pi | Claude Code | Codex | Cursor | Droid |
|---------|--------------|-----|-------------|-------|--------|-------|
| Per-task model | ✅ explicit + role + auto | ✅ | ✅ | ✅ | ✅ | ✅ |
| Recursive spawning | ✅ depth 3-5 | ✅ | ✅ depth 5 | ✅ depth 1 | ✅ | ✅ |
| Any-to-any comms | ✅ peer-to-peer | ✅ intercom | ✅ teams | ❌ | ❌ | ❌ |
| Model routing | ✅ cost-aware | ❌ | ❌ | ❌ | ❌ | ✅ factory router |
| Token budgets | ✅ per-agent | ❌ | ❌ | ❌ | ❌ | ✅ |
| Agent discovery | ✅ by cap/role | ❌ | ✅ by name | ❌ | ❌ | ❌ |

**Ion's edge:** Combines Pi's intercom model with Droid's model routing and Claude Code's recursive spawning. This is closer to SOTA than any single reference implementation.

## Phased Approach

### Phase 1: Pi Parity (Current)

**Goal:** Rock-solid core agent loop, clean architecture, Pi-level features

- Agent loop greenfield (loop.go, stream.go, tools.go) ✅
- AgentMessage Parts single source of truth ✅
- llm.Response Blocks single source of truth ✅
- Session cleanup (dead code, dependencies) ✅
- Event system, hooks, context management ✅

**Exit criteria:** All Pi parity features working, clean architecture, no debt

### Phase 2: SOTA Agent Harness (Future)

**Goal:** Multi-agent, model routing, communication, dynamic workflows

- AgentManager (recursive spawning, depth tracking)
- AgentConfig (TOML-based agent definitions)
- ModelRouter (explicit, role-based, auto)
- Intercom (bidirectional messaging, agent discovery)
- Quality hooks (BeforeSpawn, AfterTask, OnIdle)
- Deferred context engine (progressive disclosure)
- Token budget management (propagation, enforcement)

**Exit criteria:** Ion supports SOTA agent harness features

### TUI Intercom Integration

**Consideration:** Intercom-like feature between agent and TUI

**Current model:**
```
TUI ←→ Session Controller ←→ Agent Loop
         (events, commands)
```

**Future model (Phase 2):**
```
TUI ←→ Intercom ←→ Agent Loop ←→ Sub-agents
         (bidirectional)
```

**Impact on current design:**
- Current event-driven architecture is sufficient for Phase 1
- Intercom can be added later without major rework
- Key interfaces (AgentManager, Intercom, ModelRouter) are designed to be extensible
- TUI intercom would be a natural extension of the Intercom interface

**Recommendation:** Hold off on TUI intercom until Phase 2. Current design is extensible enough to support it later without major changes.

## Design Extensibility

The current architecture is designed to be extensible:

```
Session Controller (wires everything)
├── AgentLoop (pure turn sequencing)
├── AgentRecovery (overflow/retry wrapper)
├── PersistenceListener (event-driven persistence)
├── Queues (steering/follow-up)
├── Event Stream (channel-based)
│
│   Phase 2 additions:
│   ├── AgentManager (sub-agent lifecycle)
│   ├── ModelRouter (model selection)
│   ├── Intercom (bidirectional messaging)
│   └── ContextEngine (deferred context)
└── TUI Intercom (future)
```

**Key insight:** The Session Controller is the integration point. Phase 2 features are additive, not rewrites. The current agent loop, event stream, and hooks are the foundation that Phase 2 builds on.
