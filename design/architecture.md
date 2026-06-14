# Architecture

## Principles

- Zero/minimal dependencies
- Shared runtime for common primitives
- Consistent API surface across extensions
- Tight integration between extensions (workflows can use autoresearch, subagents can be workflow steps)
- Optimized for common provider tiers (local, flash, primary, pro)
- Clean, focused codebases (~200 lines per extension, not 200+ files)

## Existing Extensions to Learn From

| Extension | Source | What to take |
|-----------|--------|-------------|
| pi-subagents (nicobailon) | 2.1K stars, 200+ files | Agent definitions, chain/parallel patterns, builtin agents |
| pi-autoresearch (davebcn87) | 7K stars | Loop pattern, keep/revert, confidence scoring, hooks |
| pi-dynamic-workflows (QuintinShaw) | 35 stars, 679 tests | Model routing, journaled resume, worktree isolation, cost accounting |
| pi-compactor (nijaru) | Custom | Context management, already built |

## Shared Primitives Needed

- VM sandbox for deterministic script execution (acorn AST validation)
- Concurrency limiter (cap parallel agent runs)
- Abort signal propagation (workflow → each subagent)
- Session management (create, run, dispose)
- Display/progress rendering (widget, fullscreen overlay)
- Token/cost tracking (real, not estimated)

## Extension Boundaries

**subagents** — declarative agent definitions, chain/parallel execution, background jobs
- Agent roles (scout, researcher, planner, worker, reviewer, oracle)
- Foreground/background execution
- Intercom messaging between agents
- Model fallback chains

**workflows** — imperative JS scripts, fan-out orchestration, journaled resume
- Model writes script → VM sandbox → agent()/parallel()/pipeline()
- Model routing (small/medium/big tiers per agent)
- Worktree isolation for parallel file edits
- Save-as-command, nested composition
- Background by default with live task panel

**autoresearch** — iterative optimization loop, evaluation harness, keep/revert
- init_experiment → run_experiment → log_experiment loop
- Metric tracking with confidence scoring
- Hooks (before/after scripts)
- Dashboard widget + fullscreen overlay
- Finalize into reviewable branches

## Integration Points

- Workflows can spawn subagents
- Autoresearch can use workflows for parallel experiments
- Subagents can trigger autoresearch loops
- All share the same concurrency/session/display primitives

## What Existing Extensions Get Wrong

**pi-subagents:** Too many files (200+), overlapping concerns, 66 open issues
**pi-dynamic-workflows (michaelliv):** Missing persistence, fake cost tracking, incomplete features
**pi-dynamic-workflows (QuintinShaw):** Too much surface area (ultracode, deep-research, adversarial-review, quality stdlib) — trying to clone Claude Code's entire feature set
**pi-autoresearch:** npm audit vulnerabilities, single author

## Design Direction

Build focused, composable extensions. Each extension does one thing well. Shared runtime means common primitives are written once. No feature bloat. No Claude Code clone — just the patterns that work.

## Research-Informed Additions (from agent-research)

Patterns confirmed by SOTA research that should be incorporated:

### From Frontier Labs
- **Script-as-plan** (Claude Code): Model writes JS orchestration script, runtime executes. Enables scale beyond turn-by-turn.
- **Classifier-gated autonomy** (Cursor auto-review): Allowlist immediate, sandbox safe, classify uncertain. Not all-or-nothing.
- **Markdown agent definitions** (Claude Code, Gemini, Codex): YAML frontmatter, git-committable, inspectable.
- **Git worktrees** as multi-agent isolation primitive.

### From Papers
- **Adversarial evaluation** (Agent-as-a-Judge): Never let the agent grade its own work.
- **Peer comparison** (RAAS): Compare iterations against each other, not absolute scores.
- **Sub-sequence optimization** (Workflow-R1): Optimize at Think-Action cycle level, not token level.

### From Anthropic Harness Patterns
- Three-agent harness: Planner, Generator, Evaluator
- Sprint contracts: decompose long work into testable units
- Structured handoffs: context compaction doesn't cure coherence drift
- Trace debugging: read traces as primary debugging loop

### From Google AX
- **Event log** for journaled execution (newline-delimited JSON)
- **Single-writer controller** prevents state corruption
- **Task executor pattern** with sub-task delegation
