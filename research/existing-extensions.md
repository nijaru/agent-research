# Existing Pi Extensions — Analysis

Detailed source-level analysis lives in [agent-research/ion/](https://github.com/nijaru/agent-research/tree/main/ion).

## pi-subagents (nicobailon)

- **GitHub:** 2,187 stars, 291 forks, 66 open issues
- **What it does:** Declarative subagent delegation — chains, parallel execution, background jobs, intercom messaging
- **Builtin agents:** scout, researcher, planner, worker, reviewer, oracle, context-builder, delegate
- **Weaknesses:** 200+ source files, 66 open issues, complex codebase
- **Architecture:** TypeScript, ~200 files across src/agents, src/extension, src/runs, src/shared, src/slash, src/tui

## pi-autoresearch (davebcn87)

- **GitHub:** 7,013 stars, 414 forks
- **What it does:** Autonomous optimization loop — try idea, benchmark, keep/revert, repeat
- **Tools:** init_experiment, run_experiment, log_experiment
- **Weaknesses:** npm audit shows 3 vulnerabilities, single author

## pi-dynamic-workflows (michaelliv)

- **GitHub:** Low adoption
- **What it does:** Model writes JS scripts → VM sandbox → agent()/parallel()/pipeline()
- **Strengths:** Clean code, proper VM sandbox with AST-level determinism checks
- **Weaknesses:** Prototype — no persistence, fake token budget (len/4), worktree isolation parsed but not implemented

## pi-dynamic-workflows (QuintinShaw)

- **GitHub:** 35 stars, 9 forks
- **What it does:** Production-ready fork of michaelliv's
- **Added:** Real model routing, journaled resume, worktree isolation, real cost accounting, background by default, /workflows TUI, quality patterns (verify, judgePanel, loopUntilDry, completenessCheck), 679 tests
- **Weaknesses:** Too much surface area — trying to clone Claude Code's entire feature set

## pi-compactor (nijaru)

- **What it does:** Context management extension — gives LLM a compact tool
- **Status:** Already built, in use

## Key Takeaways

| Extension | Take | Avoid |
|-----------|------|-------|
| pi-subagents | Agent definitions, chain/parallel patterns | 200+ files, overlapping concerns |
| pi-autoresearch | Loop pattern, keep/revert, hooks | npm vulnerabilities, single author |
| QuintinShaw's workflows | Model routing, journaled resume, worktrees | Feature bloat, Claude Code clone |
| michaelliv's workflows | VM sandbox, acorn AST validation | Fake cost tracking, incomplete |
| pi-compactor | Already built | — |
