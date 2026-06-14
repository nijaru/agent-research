# Workflow-R1: Group Sub-sequence Policy Optimization for Multi-turn Workflow Construction

**Paper:** arXiv:2602.01202 (Feb 2026)
**Authors:** Mingze Kong, Zikun Qu et al.

## Core Idea

Reformulates workflow construction as a **multi-turn, natural language-based sequential decision-making process** instead of one-shot code generation. Introduces GSsPO (Group Sub-sequence Policy Optimization) — a structure-aware RL algorithm for multi-turn agentic reasoning.

## Problem with Existing Approaches

Current workflow optimization methods generate complete workflow programs (as code) in a **single pass**, before any operator is executed. This is the "Static Execution Trap":
- Planning decoupled from runtime execution
- Excessive constraints on model's coding capabilities
- No flexibility for dynamic problem-solving

## GSsPO — The Key Algorithm

**Problem:** RL methods for LLMs operate at either token level (GRPO) or full sequence level (GSPO). Both are wrong for agentic workflows:
- Token-level: disrupts semantic integrity (treats complex reasoning as independent tokens)
- Sequence-level: obscures distinct decision steps

**Solution:** Optimize at the **composite sub-sequence** level — the atomic Think-Action cycle. Align gradient updates with semantic boundaries of multi-turn interactions.

## How It Works

1. Agent engages in interleaved **Think-Act-Observe cycles**
2. Each decision conditioned on intermediate operator execution results
3. Natural language simplifies optimization (no syntactic precision needed)
4. RL training without expert demonstrations (cold-start learning)

## Results

- Outperforms competitive baselines on multiple QA benchmarks
- HotpotQA: 63.3 EM
- 2WikiMultiHopQA: 43.5 EM
- Bamboogle: 57.6 EM

## Relevance to Pi Extensions

### Key Insight: Multi-Turn > One-Shot for Workflows

Workflow-R1 validates the intuition behind pi's dynamic workflows: **iterative, multi-turn workflow construction outperforms single-pass code generation**. The model should be able to observe intermediate results and adjust the plan.

### Pattern: Think-Act-Observe Cycles

This is exactly what pi's workflow agent() calls enable — each agent call is a Think-Act cycle, and the model observes results before deciding next steps. Workflow-R1 provides theoretical grounding for this pattern.

### Implications for Autoresearch

GSsPO's sub-sequence optimization could inform how autoresearch evaluates and selects between iterations. Instead of keep/revert based on single metrics, optimize at the "iteration" level.

### Limitations

- Research paper, not a production system
- RL training requires significant compute
- Results on QA benchmarks, not coding tasks specifically
- Cold-start learning without demonstrations means initial quality may be low
