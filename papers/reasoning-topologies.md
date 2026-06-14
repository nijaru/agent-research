# Reasoning Topologies: Chain, Tree, Graph of Thought

## The Landscape

A family of papers exploring how to structure LLM reasoning beyond linear chains. The core insight: **human thought is non-linear**, and reasoning quality improves when the structure matches the problem.

## Key Papers

### Chain of Thought (CoT) — Wei et al. 2022
- Sequential step-by-step reasoning
- Foundation for everything else
- Simple but limited: no backtracking, no branching

### Tree of Thoughts (ToT) — Yao et al. 2023
- Branching reasoning paths with exploration/backtracking
- BFS/DFS over thought tree
- Better for problems requiring search (puzzles, planning)
- More expensive: multiple LLM calls per step

### Graph of Thoughts (GoT) — Besta et al. 2024
- Arbitrary graph structures, not just trees
- Thoughts can be refined, aggregated, split
- Most flexible but most complex
- Significant accuracy improvements (e.g., 87.59% on ScienceQA with T5-base)

### Framework of Thoughts (FoT) — 2025
- **Foundation framework** for building and optimizing dynamic reasoning schemes
- Implements ToT, GoT, and ProbTree within single framework
- Built-in: hyperparameter tuning, prompt optimization, parallel execution, intelligent caching
- Shows significantly faster execution, reduced costs, better task scores

### Adaptive Graph of Thoughts (AGoT) — 2025
- **Test-time adaptive reasoning** — no training required
- Recursively decomposes queries into DAG of subproblems
- Selectively expands only subproblems needing analysis
- Unifies chain, tree, and graph paradigms
- Up to 46.2% improvement on GPQA (comparable to RL approaches)

## The Taxonomy (from Demystifying paper, ETH Zurich)

Besta et al. created the first taxonomy of structure-enhanced LLM reasoning:

**Key dimensions:**
- **Structure type**: chain → tree → graph → dynamic
- **Representation**: how the structure is encoded in context
- **Algorithm**: how the model traverses/exploits the structure
- **Topology**: spatial representation within context window

**Fundamental finding:** More complex structures (graphs > trees > chains) generally perform better but cost more. The optimal structure depends on task complexity.

## Relevance to Pi Extensions

### Direct Mapping to Workflow Patterns

| Reasoning Topology | Pi Equivalent |
|-------------------|---------------|
| Chain of Thought | Sequential agent() calls |
| Tree of Thoughts | Parallel branches with evaluation |
| Graph of Thoughts | DAG workflow with dependencies |
| Adaptive Graph | Dynamic workflow with replanning |

### Pattern: Structure Selection Matters

The key insight isn't "use the most complex structure" — it's **match the structure to the problem**. Simple tasks need chains. Complex tasks need graphs. Pi should make it easy to use the right topology.

### FoT's Practical Features

FoT's built-in features map directly to what pi needs:
- **Hyperparameter tuning** → autoresearch optimization
- **Prompt optimization** → DSPy-style compilation
- **Parallel execution** → workflow fan-out
- **Intelligent caching** → session persistence

### AGoT's No-Training Approach

AGoT achieves RL-comparable improvements **at test time only** — no training required. This is the right model for pi (we use API models, can't train them). Dynamic decomposition at inference time.

## What Doesn't Work

- **Static structures** — pre-defining the reasoning topology doesn't adapt to problem difficulty
- **Full graph exploration** — too expensive, need selective expansion
- **Token-level optimization** — disrupts semantic integrity (same finding as Workflow-R1)

## Key Takeaway

The reasoning topology literature validates the **script-as-plan** pattern: the model should choose and execute a reasoning structure dynamically, not be locked into a fixed approach. Pi's workflow system already does this — the model writes the structure as code.
