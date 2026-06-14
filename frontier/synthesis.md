# Frontier Synthesis — Cross-Cutting Patterns

## Convergent Patterns (everyone is doing this)

### 1. Git Worktrees as Multi-Agent Isolation

Every tool converged on git worktrees within the same 6-week window (Feb-Apr 2026):
- **Cursor**: 8 parallel agents, each in own worktree
- **Claude Code**: worktree isolation for workflow subagents
- **Windsurf**: 5 parallel agents in worktrees
- **Gemini CLI**: DAG orchestration with worktree isolation (in progress)

Worktrees existed since Git 2.5 (2015), used by <5% of developers. AI agents made them architecturally necessary.

### 2. Markdown-Based Agent Definitions

- **Claude Code**: `.claude/agents/*.md` with YAML frontmatter
- **Gemini CLI**: `.gemini/agents/*.md` with YAML frontmatter
- **Codex CLI**: Custom agent files with model configs and instructions
- **Cursor**: `.cursor/agents`

The pattern: simple Markdown files, git-committable, inspectable, with YAML metadata for model/tools/permissions.

### 3. Context Isolation as First-Class Primitive

The fundamental insight across all tools: **subagents run in their own context windows, return only summaries**. This is what enables scale. Without it, context fills up, quality degrades, costs explode.

### 4. Explicit > Automatic Spawning

- **Codex**: Only spawns when explicitly asked (conservative)
- **Claude Code**: Automatic but gated by ultracode setting
- **Cursor**: User-initiated from Agents Window

Trend: start explicit, add automatic as trust builds.

### 5. Background/Async Execution

All tools moving toward: dispatch task → continue working → review when done. The "watch the agent work" model is dying.

## Divergent Approaches

### Orchestration Model

| Tool | Approach | Who holds the plan |
|------|----------|-------------------|
| Claude Code | Script-as-plan (JS workflows) | The script |
| Codex CLI | Turn-by-turn with explicit spawning | The model |
| Gemini CLI | DAG with Plan/Tracker separation | The DAG |
| Cursor | Visual fleet management | The user |
| Windsurf | Kanban + Devin | The cloud agent |

### Persistence Model

| Tool | Approach |
|------|----------|
| Codex | SQLite-backed threads (survives restarts) |
| Claude Code | Session-based, resumable within session |
| Gemini CLI | Session checkpointing, file-based state |
| Cursor | Git branches as persistence |

### Scale Ceiling

| Tool | Max parallel agents |
|------|-------------------|
| Claude Code | 1,000 per workflow run |
| Cursor | 8 simultaneous |
| Windsurf | 5 simultaneous |
| Codex | No explicit limit, explicit-only spawning |
| Gemini CLI | Not yet specified |

## What Actually Works (Field Reports)

1. **Codebase audits** — parallel search + adversarial verification. This is the killer use case for fan-out.
2. **Large migrations** — 500-file framework swaps. Each agent handles a subset.
3. **Multi-angle analysis** — "tear apart this plan from investor, customer, competitor perspectives"
4. **Overnight cloud agents** — dispatch complex task, review PR in morning

## What Doesn't Work Yet

1. **Automatic merge coordination** — no tool handles cross-agent file conflicts well
2. **Cost control** — no hard budget limits, only agent count caps
3. **Quality variance** — fan-out can amplify errors as easily as catch them
4. **Subagent orchestration by subagents** — nesting is immature everywhere
5. **Model routing** — most tools don't dynamically route based on task complexity

## Token Economics Summary

- Fan-out pays for itself when task is **wider than one context window**
- Break-even point: ~5-10 files of parallel exploration
- Not worth it for: single-file edits, quick lookups, sequential dependencies
- **Cost amplification risk**: a single workflow run can be 10-100x a normal session
- No tool has solved the "hard cost budget" problem yet

## Anthropic's Harness Patterns (June 2026)

Anthropic published definitive guidance on long-running agent architecture. Key patterns:

### The Three-Agent Harness: Planner, Generator, Evaluator
- **Planner**: Decomposes task into testable sprint contracts
- **Generator**: Executes work against contracts
- **Evaluator**: Adversarial review (NOT self-evaluation — self-evaluation is a trap)
- Contracts must be specific enough to grade with rubrics LLMs can apply

### Context Engineering (six patterns that work)
1. **Context rot**: Quality degrades as context grows (n² attention). Compression, not accumulation.
2. **Position rule**: Query at end, not beginning. Improves recall.
3. **Structured handoffs**: Context compaction doesn't cure coherence drift — structured handoffs do.
4. **Generator-evaluator separation**: Never let the agent grade its own work.
5. **Sprint contracts**: Decompose long work into testable units.
6. **Trace debugging**: Read traces as primary debugging loop.

### The API Surface (Claude Code Workflows)
- `agent(prompt, opts?)` — spawn subagent with model, isolation, output schema
- `parallel([fns])` — parallel orchestration
- Standard JS built-ins (JSON, Math, Array) for processing
- Six patterns: classify-and-act, map-reduce, adversarial review, iterative refinement, tree exploration, multi-perspective analysis

### Failure Modes Confirmed by Anthropic
1. **Agentic laziness**: Agent declares task finished halfway through ("I checked 20 of 50 items, all good")
2. **Self-preferential bias**: Agent verifying its own work always says it's good
3. **Goal drift**: After context compression, original constraints get lost
4. **Context anxiety**: Agent starts wrapping up early as window fills

## Framework Status (June 2026)

### LangGraph — Production Default
- v1.0 (Oct 2025), v1.2 (May 2026)
- 34% of agent-framework citations in production architecture docs at 1000+ employee companies
- 90M monthly downloads, 28K GitHub stars
- Production at Klarna, Uber, JP Morgan, BlackRock, Cisco, LinkedIn
- Best for: stateful, checkpointed, graph-based orchestration
- Complex task success rate: 62%, Medium: 76%, Simple: 88%

### CrewAI — Role-Based Prototyping
- v0.105 (Mar 2026), 47.8K GitHub stars, 60% Fortune 500 adoption
- Best for: rapid prototyping, role-based agent teams
- Lower learning curve, faster setup (2-3 engineer-days vs 10-14 for LangGraph)
- Complex task success rate: 54%

### AutoGen — Maintenance Mode
- Microsoft shifted to broader Microsoft Agent Framework
- 55K GitHub stars, community packages still work
- New projects should look elsewhere in 2026

### Vanilla SDK Approach (growing trend)
- Octomind ripped LangChain in 2024, replaced with direct SDK calls
- Pattern: direct SDK + small state machine, ship faster
- 61% of enterprises adopted agent frameworks (up from 18% in 2024)
- But many teams report framework → direct SDK migration after month 3

## Implications for Pi Extensions

### Must-Have Primitives

1. **Context isolation** — subagents in own windows
2. **Git worktree lifecycle** — add, register, cleanup
3. **Markdown agent definitions** — YAML frontmatter, git-committable
4. **Script-as-plan** — model writes orchestration code
5. **Journaled execution** — survive interruption, resume
6. **Model routing** — small/medium/big tiers per agent role
7. **Token/cost tracking** — real, not estimated
8. **Classifier-gated autonomy** — Cursor's auto-review pattern, allowlist/sandbox/classify
9. **Adversarial evaluation** — separate evaluator, never self-evaluation

### Differentiators to Target

1. **Hard cost budget** — no one has this, everyone needs it
2. **Composable extensions** — workflows can use autoresearch, subagents can be workflow steps
3. **Clean codebase** — Claude Code is 200+ files, aim for ~200 lines per extension
4. **Open source** — Cursor/Windsurf are closed, Claude Code partially open, Codex open
5. **Lightweight** — Go binary (Antigravity) or single executable (Codex Rust), not 200+ file TypeScript

### Anti-Patterns to Avoid

1. **Feature bloat** — QuintinShaw's fork trying to clone Claude Code's entire feature set
2. **Fake cost tracking** — michaelliv's len/4 token estimation
3. **Overlapping concerns** — pi-subagents' 200+ files with 66 open issues
4. **Single-author risk** — pi-autoresearch has 3 npm vulnerabilities
5. **Self-evaluation** — agents grading their own work (confirmed trap by Anthropic)
6. **Context cramming** — stuffing everything into one window instead of isolating
