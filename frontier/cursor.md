# Cursor — Agent Orchestration Analysis

## Architecture Overview

Cursor 3 (April 2026) rebuilt the entire interface around agent orchestration. $29.3B valuation. The most explicit statement of "IDE as fleet orchestrator."

### Agents Window

- Replaces Composer panel as primary interface
- Unified sidebar showing all active agent sessions (local + cloud)
- Agents from mobile, web, desktop, Slack, GitHub, Linear all appear in one place
- Click into any session for chat history and file diffs
- `Cmd+Shift+P` → "Agents Window"

### Parallel Execution

- Up to **8 agents simultaneously** on a single codebase
- Each agent operates in **isolated git worktree**
- `/worktree` command runs `git worktree add`, spawns scoped agent process
- Agents register in Agents Window with status, branch, current step
- `/best-of-n` runs same prompt across multiple models, compares outputs

### Background Agents (Cloud)

Three execution modes:
1. **Local agents** — Composer 2 model, in open workspace, 5-15 second round trips
2. **Cloud agents** — isolated Ubuntu VM, clones repo, creates branch, writes code, runs tests, opens PR
3. **Local-to-cloud handoff** — move session between targets mid-task

Cloud agents include:
- Full desktop with browser access (Computer Use)
- Video recording of feature working
- Screenshots as proof
- BugBot review before human review

### Self-Driving Codebases (Research)

Cursor's internal research ran **thousands of agents** building a web browser:
- Rust-based harness on single machine
- Agents spawned for tasks, nudged when stopped
- Dynamic spawning based on dependency graph
- GPT-5.1/5.2 for long-running agent instruction following
- 35% of Cursor's own merged PRs written by autonomous cloud agents

### Subagents

- Defined in `.cursor/agents`
- Custom agents with different model configurations
- Explorer subagent for codebase search
- Community reports: orchestrator managing multiple subagents in parallel still has issues (agents getting stuck)

## What Works

1. **Git worktrees as isolation primitive** — solves the single-checkout bottleneck, each agent gets own directory/branch/build
2. **Visual orchestration** — Agents Window makes multi-agent management accessible
3. **Cloud execution with proof** — video + screenshots + PR, not just code diffs
4. **Local-to-cloud handoff** — start quick, escalate to cloud when task grows
5. **Model racing** (`/best-of-n`) — hedge across models

## Failure Modes

- **No automatic merge coordination** — two agents can write to same file, coordination is user's job
- **No ordering enforcement** between sessions
- **3.0 shipped with rough edges** — 5 patches in 5 days (segfaults, session persistence, missing features)
- **Subagent orchestration still immature** — community reports agents getting stuck when orchestrator tries to manage them in parallel
- **`cursor --classic`** escape hatch needed on day one — aggressive default change

## Token Economics

- 8 parallel agents = 8x token consumption
- Cloud agents have usage limits tied to plan
- Composer 2 model (Cursor's own) comes with higher limits than third-party models
- Background agents: Pro plan ($20/month) + GitHub connection

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Agents Window as primary UI | Most code will be written by AI agents |
| Git worktrees for isolation | Native Git feature, no custom filesystem needed |
| Cloud VMs with video proof | Trust through verification |
| 8-agent limit | Balance between parallelism and resource contention |
| `--classic` escape hatch | Acknowledge not all workflows are agent-first yet |

## API Surface for Pi Extensions

Key primitives:
- Git worktree lifecycle (add, register, cleanup)
- Agent session management (local ↔ cloud handoff)
- Video/screen recording as artifact
- `/best-of-n` model racing pattern
- Visual agent dashboard

## June 2026 Updates (Cursor 3.5–3.7)

Cursor shipped 3 major versions in 6 weeks. Key changes:

**Cursor 3.5 (May 20):**
- Shareable canvases — interactive artifacts (reports, dashboards) shareable via link
- `/loop` skill for iterative workflows
- Automations in Agents Window (multi-repo + no-repo)
- **Composer 2.5** — based on Moonshot's Kimi K2.5 open weights (1.04T params, 32B active)
  - Rivals GPT-5.5 coding performance at fraction of cost
  - $0.50/$0.20/$2.50 per million input/cached/output tokens
  - RL-trained with simulated agentic harness matching Cursor CLI

**Cursor 3.6 (May 29):**
- **Auto-review run mode** — the biggest autonomy change:
  - Allowlisted calls run immediately
  - Sandboxable calls run in sandbox
  - Everything else goes to a **classifier subagent** that decides: allow, reroute, or ask for approval
  - Classifier is a small, fast model with enough reasoning to judge safety
  - Works on shell, MCP, and fetch tool calls
  - Key insight: lower-reasoning models weren't always faster — sometimes spent more tokens searching for worse answers
- Organizations for Enterprise (Org → Team → Groups)

**Cursor 3.7 (June 5):**
- Design Mode: multi-select UI elements, voice narration while agent runs
- Canvas Design Mode: annotate directly inside canvases
- **Cursor SDK**: nested subagents, custom tools via `local.customTools` (no MCP server needed), auto-review classifier, JSONL/SQLite/custom stores
- Interactive context-usage report

**BugBot improvements (June 2026):**
- 3x faster, 22% cheaper, 10% more bugs found
- 90% of runs finish under 3 minutes
- `/review` runs BugBot + Security Review before pushing
- Syncs with GitHub/GitLab — recognizes already-reviewed diffs
- Powered by Composer 2.5
- Incremental review option (only new changes since last review)

**Pricing changes:**
- Standard seat: $32/mo annual, $40/mo monthly (increased usage)
- Premium seat: $96/mo annual, $120/mo monthly (5x usage at 3x cost)
- Split pools: Composer+Auto vs Third-Party API
- Real-time usage visibility in dashboard

**Current version**: 3.7 (June 5, 2026)

## Sources

- https://cursor.com/blog/self-driving-codebases
- https://agentmarketcap.ai/blog/2026/04/05/cursor-april-2026-agent-mode-overhaul-background-agents-ide-convergence
- https://www.studiomader.it/blog/cursor-3-agentic-architecture.html
- https://www.infoq.com/news/2026/04/cursor-3-agent-first-interface/
- https://bytewaves.news/tutorials/how-to-use-cursor-background-agents-branching-prs-parallel-runs-2026/
- https://cozypet.github.io/cursor-cloud-harness/
- https://cursor.com/blog/bugbot-updates-june-2026
- https://cursor.com/blog/agent-autonomy-auto-review
- https://cursor.com/blog/design-mode
- https://www.aiwerse.blog/ai-tools/cursor-just-got-smarter-what-s-new-in-cursor-3-7-update
- https://cursor.com/blog/teams-pricing-june-2026
- https://www.deeplearning.ai/the-batch/cursor-fits-its-model-to-its-agent
- https://developertoolkit.ai/en/cursor-ide/version-management/
