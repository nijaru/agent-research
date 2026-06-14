# Codex CLI — Agent Orchestration Analysis

## Architecture Overview

OpenAI's coding agent, rewritten from TypeScript to Rust. Open-source. Three surfaces (CLI, VS Code, macOS app) powered by same harness via JSON-RPC App Server.

### Session Model

Three primitives:
- **Thread**: Persistent conversation backed by SQLite. Survives restarts, can be resumed/forked/archived/rolled back. *Fundamental difference from Claude Code.*
- **Turn**: One round-trip cycle (user → model → tool calls → results → model)
- **Item**: Granular events within a turn (messages, shell output, file edits, reasoning traces)

### Subagent System

- Codex only spawns subagents when **explicitly asked** (not automatic)
- Each subagent gets its own model and tool work
- Built-in agents: `default` (general-purpose), `worker` (execution-focused)
- Custom agents via Markdown files with different model configs and instructions
- Subagents inherit parent's sandbox policy and approval mode
- Live runtime overrides propagate to children

**Orchestration:**
- Codex handles spawning, routing, waiting, closing
- Consolidated response after all agents complete
- Approval requests can surface from inactive threads
- `/agent` to switch between active threads

### App Server (JSON-RPC)

- Bidirectional protocol, designed for multi-client (CLI, VS Code, JetBrains, Xcode, desktop app)
- First tried MCP, but MCP semantics didn't map cleanly to rich agent interactions (streaming, approval, diffs)
- Built custom JSON-RPC protocol that mirrors the TUI loop
- Stable, backward-compatible API surface

### Codex Cloud

- Each task runs in isolated cloud sandbox, preloaded with repo
- Tasks: read/edit files, run commands, tests, linters
- 1-30 minutes per task
- Verifiable evidence: terminal logs, test outputs
- Guided by AGENTS.md files in repo

## What Works

1. **SQLite-backed persistence** — threads survive process restarts. No session loss.
2. **Explicit subagent spawning** — no accidental token waste from automatic delegation
3. **Approval propagation** — parent's interactive overrides (sandbox, yolo mode) apply to children
4. **MCP as extension port** — run Codex as MCP server, connect from Agents SDK
5. **Single binary** — Rust rewrite ships as one executable with embedded assets

## Failure Modes / Open Issues

- **#22099: Parallel-first subagent policy unclear** — no built-in policy for when to split work into subagents
- **#19197: Orphaned subagents** — lifecycle controls, session freezes
- **#17737: Background terminal monitoring** — long-running work visibility
- **Community fork "Open Codex CLI"** explores: parallel-first planning, nonblocking background execution, persistent task panel
- **No workflow scripting** like Claude Code — orchestration is turn-by-turn

## Token Economics

- Subagents consume more tokens than single-agent runs
- No built-in cost tracking or budget limits
- `exec` command for scripting/automation (non-interactive)

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| SQLite persistence | Cross-session continuity, fork/resume |
| JSON-RPC over MCP | Rich interaction patterns (streaming, approval, diffs) |
| Explicit spawning only | Cost control, user agency |
| Rust binary | Performance, cross-platform sandbox (bubblewrap) |
| Open source | Community can inspect and extend |

## API Surface for Pi Extensions

Key primitives:
- Thread persistence (SQLite-backed sessions)
- Explicit-only subagent spawning with approval propagation
- MCP server mode for composability
- `exec` command for scripted automation
- Session resume/fork from CLI

## June 2026 Updates (v0.136–v0.139)

Codex has been shipping aggressively. Key changes since April:

**Multi-agent v2** (v0.137-0.138):
- Per-thread runtime configuration — different agents in same session can use different models/temperatures/tools
- Spawned agents inherit cleaner defaults without manual override
- Path-based v2 activity tracking (canonical agent paths, not nicknames)
- `/agent` list shows running path-backed agents with summaries from bounded local event buffers

**Cross-platform handoff** (v0.138):
- `/app` command hands off CLI thread to Codex Desktop (macOS + Windows)
- `codex terminal` CLI command opens Desktop app directly
- Context, running state, and output history transfer

**Web search in code mode** (v0.139):
- Standalone web search callable from code mode, including nested JS tool calls
- Agent can fetch current docs/API surface mid-turn without breaking context

**Auth & Enterprise**:
- v2 personal access tokens (scoped, revocable) for CLI and app-server
- Monthly credit limits visible in TUI
- Cloud-managed config bundles for enterprise

**Other notable changes**:
- GPT-5.5 available in Codex (Apr 23) — recommended default for implementation, refactors, debugging
- Goal mode out of experimental (May 21) — stable
- Hooks GA (May 14)
- Chrome extension (May 7)
- Remote mobile access via ChatGPT iOS (May 14)
- Computer Use on Windows (May 29)
- Amazon Bedrock model provider (Jun 1)
- Sites plugin — deploy from Codex (Jun 2, Preview)
- Session archiving (`/archive`, `codex archive`/`codex unarchive`)
- MCP schema fixes: `oneOf`/`allOf` keywords preserved through compaction
- Usage grew from 3M weekly active developers (Apr) to 5M (Jun)

**Current version**: v0.139.0 (June 9, 2026)

## Sources

- https://openai.com/index/unlocking-the-codex-harness/
- https://openai.com/so-DJ/index/unrolling-the-codex-agent-loop/
- https://developers.openai.com/codex/subagents
- https://developers.openai.com/codex/cli/features
- https://github.com/openai/codex/issues/22099
- https://zylos.ai/research/2026-03-26-openai-codex-cli-architecture-multi-runtime-patterns
- https://github.com/openai/codex/releases/tag/rust-v0.138.0
- https://github.com/openai/codex/releases/tag/rust-v0.139.0
- https://www.developersdigest.tech/blog/codex-changelog-june-2026
- https://simoncarter.ai/posts/codex-cli-v0-139-0-code-mode-can-now-search-the-web-mid-task-and-complex-tool-sc/
- https://waytoclawearn.com/news/openai-codex-cli-1380-handoff-multi-agent-v2-2026
