# Pi Extensions Roadmap

## Research Phase — Complete

Priorities 1-3 are done. Detailed findings in [agent-research](https://github.com/nijaru/agent-research):

- **Frontier implementations**: `frontier/` — Claude Code, Codex, Gemini/Antigravity, Cursor, Windsurf
- **Academic papers**: `papers/` — RLM, DSPy, Workflow-R1, reasoning topologies, ADAS
- **Frameworks**: `frameworks/` — LangGraph, CrewAI, Google AX, Vercel AI SDK, Pydantic AI
- **Source analysis**: `ion/` — Pi, Codex, Claude Code internals

## Key Findings (from agent-research)

1. **Script-as-plan** is the winning pattern (Claude Code dynamic workflows, RLM, Workflow-R1)
2. **Git worktrees** are the multi-agent isolation primitive (everyone converged Feb-Apr 2026)
3. **Never self-evaluate** — adversarial evaluators only (Anthropic confirmed)
4. **Hard cost budget** — nobody has this, everyone needs it
5. **Classifier-gated autonomy** — Cursor auto-review, allowlist/sandbox/classify
6. **Context isolation** as first-class primitive — subagents in own windows, return summaries
7. **Markdown agent definitions** — YAML frontmatter, git-committable (Claude Code, Gemini, Codex all converge)

## Design Phase — Next

What each extension should look like, informed by the research:

### Shared Runtime

Primitives needed across all extensions:
- VM sandbox for deterministic script execution
- Concurrency limiter (cap parallel agent runs)
- Abort signal propagation (workflow → each subagent)
- Session management (create, run, dispose)
- Token/cost tracking (real, not estimated) — **hard budget as differentiator**
- Model routing (small/medium/big tiers per agent role)

### subagents extension

- Declarative agent definitions (Markdown, YAML frontmatter)
- Chain/parallel/async execution
- Fresh context by default, fork on demand
- Bounded depth (default 3, configurable to 5)
- Quality gates: BeforeSpawn, AfterTask, OnIdle hooks
- Intercom messaging between agents (pi already has this)
- Per-agent model selection (explicit, role-based)

### workflows extension

- Model writes JS script → VM sandbox → agent()/parallel()/pipeline()
- Journaled execution (event log for resume, like Google AX)
- Worktree isolation for parallel file edits
- Quality stdlib: verify(), judgePanel(), loopUntilDry(), completenessCheck()
- Save-as-command, nested composition
- Background by default with live task panel
- Hard token budget per run

### autoresearch extension

- init_experiment → run_experiment → log_experiment loop
- Metric tracking with confidence scoring
- Hooks (before/after scripts)
- Dashboard widget + fullscreen overlay
- Finalize into reviewable branches
- Adversarial evaluation (never self-evaluate)
- Peer comparison across iterations (RAAS pattern)

## Deliverables

- [ ] `design/subagents.md` — Subagent extension API design
- [ ] `design/workflows.md` — Dynamic workflow extension API design
- [ ] `design/autoresearch.md` — Autoresearch extension API design
- [ ] `design/shared-runtime.md` — Shared primitives specification
