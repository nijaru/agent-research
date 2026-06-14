# Claude Code Dynamic Workflows: Patterns, Use Cases, and Tips

**Date:** 2026-06-02
**Source:** [Thariq Shihipar (@trq212) — Anthropic](https://x.com/trq212/status/...)
**Status:** Reference material for Ion Phase 2 (Pi+)
**Related:** `rlm-dynamic-workflows.md` — RLM foundations and Ion architecture proposal

---

## Overview

Dynamic workflows let Claude Code write its own harness JavaScript on the fly, custom-built
for the task at hand. Instead of planning and executing in the same context window (which
degrades over long tasks), workflows orchestrate separate Claudes with isolated context
windows and focused goals.

Launched with Claude Opus 4.8. Workflows execute a JavaScript file with special functions
to spawn and coordinate subagents. Standard JS (JSON, Math, Array) is available for data
processing. Workflows can specify per-agent models and whether subagents run in their own
worktree.

**Key tradeoff:** Dynamic workflows often use significantly more tokens. Use them for tasks
that genuinely benefit from orchestration, not routine coding.

---

## Failure Modes That Workflows Address

| Failure Mode             | Description                                                                 |
| ------------------------ | --------------------------------------------------------------------------- |
| **Agentic laziness**     | Declares task done after partial progress (e.g., 20 of 50 security items)   |
| **Self-preferential bias** | Prefers own results when asked to verify/judge against a rubric            |
| **Goal drift**           | Gradual loss of fidelity to original objective across many turns, especially after compaction |

Workflows combat these by orchestrating separate agents with their own context windows and
isolated goals.

---

## Workflow Patterns

### 1. Classify-and-Act

Classifier agent decides task type → routes to different agents or behavior based on
classification. Can also classify at the end to determine output.

### 2. Fan-out-and-Synthesize

Split task into many smaller steps, run an agent on each, then synthesize. Useful when:
- Large number of smaller steps
- Each step benefits from a clean context window (no cross-contamination)

The synthesize step is a **barrier** — waits for all fan-out agents, then merges structured
outputs.

### 3. Adversarial Verification

For each spawned agent, run a separate agent to adversarially verify its output against a
rubric or criteria.

### 4. Generate-and-Filter

Generate N ideas → filter by rubric/verification → dedupe → return only highest quality,
tested ideas.

### 5. Tournament

Spawn N agents that each attempt the same task using different approaches. Judging agents
evaluate pairwise until a winner emerges. Better than absolute scoring for taste-based
tasks (naming, design).

### 6. Loop Until Done

For tasks with unknown work: loop spawning agents until a stop condition is met (no new
findings, no more errors in logs) instead of a fixed number of passes.

---

## Use Cases

### Migrations and Refactors
Break into per-module/per-file steps. Spin off a subagent per fix in a worktree, then
adversarial review, then merge. Tell agent to avoid resource-intensive commands to maximize
parallelism. *(Reference: Bun's Zig→Rust rewrite.)*

### Deep Research
Fan-out web searches → fetch sources → adversarially verify claims → synthesize cited
report. Also applicable to internal research (Slack status reports, codebase exploration).

### Deep Verification
One agent identifies all factual claims in a report → spin off subagent per claim to
check in detail → verification agent checks source quality.

### Sorting
For large qualitative sorts (1000+ items), use tournament, pairwise-comparison pipeline,
or bucket-rank-in-parallel then merge. Comparative judgment is more reliable than absolute
scoring. Each comparison is its own agent; deterministic loop holds the bracket.

### Memory and Rule Adherence
Create workflow with verifier agents — one per rule. Skeptic persona subagent reviews rules
to reduce false positives. Reverse: mine sessions/code review comments for recurring
corrections → cluster → adversarially verify → distill into CLAUDE.md rules.

### Root-Cause Investigation
Generate hypotheses from disjoint evidence (separate agents for logs, files, data). Each
hypothesis faces a panel of verifiers and refuters. Applicable beyond code: sales drops,
pipeline failures, post-mortems.

### Triaging at Scale
Classify each item → dedupe against tracked → attempt fix or escalate. **Quarantine
pattern:** agents reading untrusted public content are barred from high-privilege actions.
Pair with `/loop` for continuous triage.

### Exploration and Taste
Explore multiple solutions → review agent with rubric → task complete when criteria met.
Use tournament for ordering/selection.

### Evals
Spin off agents in worktrees → comparison agents grade outputs against rubric. Useful for
refining skills against specific criteria.

### Model/Intelligence Routing
Classifier agent decides which model to use based on task complexity. Research before
execution identifies best model (e.g., Sonnet for simple explain, Opus for large refactor).

---

## When NOT to Use Dynamic Workflows

- Regular coding tasks that don't need orchestration
- When you'd ask "does this really need more compute?"
- A panel of 5 reviewers is overkill for most traditional coding

---

## Practical Tips

### Prompting
- Detailed, pattern-specific prompting creates best results
- "Quick workflows" exist for small tasks (e.g., quick adversarial review of an assumption)
- Use specific pattern names in prompts

### Combining with /goal and /loop
- Pair with `/loop` for repeatable workflows (triage, research, verification)
- Pair with `/goal` to set hard completion requirements

### Token Budgets
Set explicit budgets: `"use 10k tokens"` caps the workflow.

### Saving and Sharing
- Press `s` in workflow menu to save
- Check into `~/.claude/workflows`
- Distribute via skills: put JS files in skill folder, reference in SKILL.md
- Prompt Claude to treat skill workflows as templates, not verbatim scripts

### Resumption
If interrupted (user action, terminal quit), resuming the session lets the workflow pick up
where it left off.

---

## Relevance to Ion

Dynamic workflows are the canonical SOTA reference for Ion's Phase 2 orchestration layer.
The patterns above map directly to Ion's proposed `ion/workflow` package:

| Claude Code Pattern     | Ion Equivalent (proposed)                                    |
| ----------------------- | ------------------------------------------------------------ |
| Fan-out-and-synthesize  | Parallel child sessions via `ion/session` + barrier merge    |
| Adversarial verification| Separate reviewer agent per child session                    |
| Tournament              | Deterministic bracket runner with pairwise comparison agents |
| Loop until done         | Orchestrator loop with stop-condition evaluation             |
| Quarantine (triage)     | Untrusted-read agents barred from write tools                |

**Key architectural insight:** Claude Code's approach of "write a JS file as the plan"
aligns with Ion's proposed `execute_workflow_script` tool. The script holds state,
dependencies, and orchestration logic outside the agent's context window.

**What to steal first for P1:**
- Adversarial verification as a built-in tool pattern (already natural in pi-subagents)
- Token budget enforcement per workflow
- Resumption semantics (Ion's durable session history should already handle this)

---

## Example Prompts (verbatim from Anthropic)

> "This test fails maybe 1 in 50 runs. Set up a workflow to reproduce it, form theories
> and adversarially test them in worktrees /goal don't stop until one theory works."

> "Using a workflow, go through my last 50 sessions and mine them for corrections I keep
> making and turn the recurring ones into CLAUDE.md rules"

> "Take my business plan and run a workflow where different agents tear it apart from an
> investor's, a customer's, and a competitor's perspective."

> "Here's a folder of 80 resumes, use a workflow to rank them for the backend role and
> double-check the top ten. Interview me using the AskUserQuestion tool for a rubric."

> "I need a name for this CLI tool. Use a workflow to brainstorm a bunch of options and
> run a tournament to pick the top 3."

> "Go through my blog post draft and using a workflow verify every technical claim against
> the codebase, I don't want to ship anything wrong."
