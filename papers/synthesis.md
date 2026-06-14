# Papers Synthesis — Key Findings for Pi Extensions

## The Five Papers That Matter Most

### 1. RLM — Context as Environment
**Takeaway:** Don't cram context into the window. Store it externally, let the model query it.
**Pi application:** Compactor, workflow state management.
**Pattern:** REPL environment with recursive sub-calls.

### 2. DSPy — Programs Over Prompts
**Takeaway:** Declarative programs with automatic optimization beat hand-crafted prompts.
**Pi application:** Autoresearch optimization backend, agent definition as compilable programs.
**Pattern:** Signature → Module → Optimizer → Compiled program.

### 3. Workflow-R1 — Multi-Turn Workflow Construction
**Takeaway:** Iterative Think-Act-Observe cycles beat one-shot workflow generation.
**Pi application:** Validates script-as-plan pattern, informs autoresearch iteration design.
**Pattern:** GSsPO at sub-sequence (Think-Action cycle) level.

### 4. ADAS — Automated Agent Design
**Takeaway:** A meta-agent can discover better agent designs than humans.
**Pi application:** Autoresearch could optimize the agent program itself, not just task output.
**Pattern:** Meta Agent Search with growing archive of discoveries.

### 5. Agent-as-a-Judge — Agentic Evaluation
**Takeaway:** Evaluating agents requires agents (intermediate feedback, tool use, planning).
**Pi application:** verify() and judgePanel() should use agentic evaluation, not single LLM calls.
**Pattern:** Planning + tool-augmented verification + persistent memory.

## Cross-Cutting Themes

### Theme 1: Structure Selection is Dynamic

The reasoning topology papers (CoT → ToT → GoT → AGoT) show that **the optimal reasoning structure depends on the problem**. FoT and AGoT unify these into frameworks that select structure dynamically.

**For pi:** Don't lock into one workflow pattern. The model should choose chains, trees, or graphs based on task complexity. The script-as-plan pattern already enables this — the model writes whatever structure it needs.

### Theme 2: Evaluation Stability is Critical

RAAS, Agent-as-a-Judge, and Workflow-R1 all point to the same problem: **how do you know if an iteration was actually better?** Solutions:
- Peer comparison instead of absolute scores (RAAS)
- Intermediate feedback instead of final-outcome-only (Agent-as-a-Judge)
- Sub-sequence optimization instead of token/sequence level (Workflow-R1)

**For pi:** Autoresearch needs principled evaluation. Single metric + keep/revert is insufficient.

### Theme 3: Design Knowledge is Transferable

ABSTRAL shows SKILL.md documents transfer across domains. ADAS shows agent designs discovered in one domain work in others. DSPy shows compiled programs generalize.

**For pi:** Invest in making agent designs inspectable and reusable. The SKILL.md format is the right abstraction.

### Theme 4: The Cost-Quality Frontier

Every paper faces the same tradeoff:
- RLM: recursive calls can be expensive (high variance)
- DSPy: compilation requires training data and compute
- ToT/GoT: more complex structures cost more
- ADAS: meta-search requires many iterations
- Multi-agent: 26% turn efficiency (ABSTRAL)

**For pi:** Hard cost budgets are essential. Token economics should be a first-class concern, not an afterthought.

## What the Papers DON'T Tell Us

1. **Production failure modes** — papers evaluate on benchmarks, not real codebases
2. **Human-in-the-loop patterns** — when to interrupt, when to proceed autonomously
3. **Composability** — how to combine these patterns in practice (workflows + autoresearch + subagents)
4. **Token budget optimization** — given a fixed budget, how to allocate across agents/iterations

These are the gaps pi can fill.

## Recommended Reading Order

1. **RLM** — understand the context-as-environment pattern
2. **Workflow-R1** — understand multi-turn workflow construction
3. **ADAS** — understand automated agent design
4. **Agent-as-a-Judge** — understand agentic evaluation
5. **DSPy** — understand program optimization (heaviest, most framework-oriented)
6. **Reasoning topologies** — understand structure selection (survey, broad)
