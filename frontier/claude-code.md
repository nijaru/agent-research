# Claude Code — Agent Orchestration Analysis

## Architecture Overview

Claude Code has the most mature multi-agent orchestration of any coding agent as of June 2026. Three layers of increasing power:

### 1. Subagents

- Each subagent runs in its **own context window** with custom system prompt, tool access, and independent permissions
- Built-in types: **Explore** (Haiku, read-only), **Plan** (research), **General-purpose**
- Custom subagents defined via Markdown files with YAML frontmatter
- Inherit parent conversation's permissions (Explore/Plan skip CLAUDE.md for speed)
- Model routing: can assign cheaper models (Haiku) to exploration, expensive to implementation

**Key pattern:** Subagents return only summaries, keeping the main context clean. This is the fundamental insight — context isolation as a first-class primitive.

### 2. Agent Teams

- A **lead agent** supervising peer sessions
- Shared task list for coordination
- Teammates keep running if lead is interrupted
- Scale: handful of long-running peers

### 3. Dynamic Workflows (shipped May 2026, GA)

The headline feature. Claude writes a **JavaScript orchestration script** that a runtime executes.

**Entry points:**
- Say `ultracode` in prompt → single task as workflow
- `/effort ultracode` → Claude decides per-task whether to use workflows
- Direct request: "Create a workflow for X"

**Architecture:**
- Script holds the plan, branching, loops, and intermediate results
- Claude's context holds only the final answer
- Runtime executes in background, session stays responsive
- Resumable within same session
- Cap: 1,000 agents per run

**Under the hood (per API analysis):**
- Uses `xhigh` effort level for deep planning
- **Mid-conversation system messages** (new Opus 4.8 capability): inject new instructions/permissions mid-task, granting orchestrator standing permission to launch workers
- Not a new API effort level — it's `xhigh` + mid-conversation system messages

**Quality patterns built in:**
- `verify()` — adversarial fact-checking
- `judgePanel()` — score N candidates, return best
- `loopUntilDry()` — keep finding until rounds stop yielding
- `completenessCheck()` — "what's missing" critic

**When to use workflow vs subagents vs skills:**

| | Subagents | Skills | Agent Teams | Workflows |
|---|---|---|---|---|
| Who decides next | Claude, turn by turn | Claude, following prompt | Lead agent, turn by turn | The script |
| Where results live | Claude's context | Claude's context | Shared task list | Script variables |
| What's repeatable | Worker definition | Instructions | Team definition | Orchestration itself |
| Scale | Few per turn | Same | Handful | Dozens to hundreds |
| Interruption | Restarts turn | Restarts turn | Teammates keep running | Resumable |

## What Works

1. **Context isolation** is the key primitive. Subagents doing research in their own window, returning only summaries.
2. **Moving the plan into code** (workflow script) enables scale beyond what turn-by-turn orchestration can handle.
3. **Adversarial verification** before reporting results — independent agents review each other.
4. **Model routing** — cheap models for exploration, expensive for synthesis.
5. **Resumability** — journaled execution survives interruption.

## Failure Modes

- **Token cost explosion** — "substantially more tokens than typical session, start scoped"
- **No hard cost limit** — only agent count cap (1,000), not token/cost cap
- **Ultracode confusion** — two different behaviors share one name (keyword trigger vs effort setting)
- **Quality still variable** — early adopters report needing to scope tasks carefully

## Token Economics

- Single workflow run can consume orders of magnitude more tokens than a normal session
- Fan-out pays for itself on tasks wider than one context window (500-file migration, codebase audit)
- Not worth it for "a few delegated lookups" — use subagents instead

## API Surface for Pi Extensions

Key primitives to steal:
- `agent(prompt, opts)` with model routing (small/medium/big tiers)
- `parallel(tasks)` for fan-out
- `pipeline(items, ...stages)` for sequential processing
- Script-as-plan pattern (model writes JS, runtime executes)
- Mid-conversation capability injection
- Quality stdlib: verify, judgePanel, loopUntilDry, completenessCheck

## June 2026 Updates

- **v2.1.154** (May 28): Opus 4.8 default, dynamic workflows GA, `/workflows` command
- **v2.1.160** (early June): `workflow` keyword renamed to `ultracode` as trigger + effort level
- **v2.1.172-173** (June): Safe Mode (`--safe-mode` disables all customizations for troubleshooting), `/cd` for mid-session directory change, doubled rate limits, per-category `/usage` breakdown
- **Parallel Bash fix**: Each tool call in a batch now returns independently — no more cascading failures from one bad command killing the whole batch. Critical for workflow reliability.
- **OTEL team-level metrics**: Observability for team usage patterns
- **Safe Mode**: `CLAUDE_CODE_SAFE_MODE=1` disables CLAUDE.md, skills, plugins, hooks, MCP servers, custom commands, agents, workflows, themes, keybindings. Admin policy settings still apply.

**Current version**: v2.1.173 (June 2026)

## Sources

- https://code.claude.com/docs/en/workflows
- https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
- https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
- https://code.claude.com/docs/en/sub-agents
- https://www.developersdigest.tech/blog/ultracode-effort-level-explained
- https://simoncarter.ai/posts/claude-code-workflow-renamed-to-ultracode-parallel-bash-bug-fixed-and-otel-team-/
- https://jangwook.net/en/blog/en/claude-code-june-2026-new-features-changelog-developer-guide/
- https://github.com/anthropics/claude-code/releases/tag/v2.1.154
