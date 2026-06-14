# Source-Level Analysis

Deep dives into Pi, Codex, and Claude Code internals. Extracted from [ion](https://github.com/nijaru/ion)/ai/ as general-purpose agent research.

## What's Here

| File | What It Analyzes | Key Finding |
|------|-----------------|-------------|
| `multi-agent-architecture-2026-06.md` | Pi vs Claude Code vs Codex vs Cursor vs Droid vs Amp | Comparison matrix; Pi has best comms, Codex has best persistence |
| `pi-agent-loop-deep-dive.md` | Pi's 742-line agent-loop.ts | Three layers: sequencing → types → stateful wrapper |
| `codex-agent-loop-analysis.md` | Codex thread-based architecture | SQLite persistence, auto-compaction, transport fallback |
| `claude-code-tool-execution-analysis.md` | Claude Code internals | Partitioned execution, permission system, streaming tools |
| `google-ax-runtime-review-2026-05.md` | Google AX distributed runtime | Event log, single-writer controller, task executor |
| `claude-code-dynamic-workflows-2026-06.md` | Thariq's workflow patterns | Six patterns: classify-and-act, map-reduce, adversarial, etc. |
| `rlm-dynamic-workflows.md` | RLM vs dynamic workflows | RLM = context processing (sequential), DW = task decomposition (parallel) |

## Relationship to ion/

These files are copies of general-purpose analyses from ion's `ai/review/` and `ai/research/` directories. Ion-specific files (correctness audits, parity analysis, performance proofs, specs) remain in ion/ai/.

If you update a file here, also update the source in ion/ai/ (or vice versa). They should stay in sync.
