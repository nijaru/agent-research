# DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines

**Paper:** arXiv:2310.03714 (Oct 2023, updated through 2026)
**Authors:** Omar Khattab et al., Stanford NLP
**Code:** https://github.com/stanfordnlp/dspy (35K stars, 410 contributors)
**Status:** Actively developed, v3.3.0b1 (May 2026)

## Core Idea

DSPy replaces hand-crafted prompt templates with **declarative Python programs** where LM calls are parameterized modules that can be **automatically optimized** against a metric.

**Key insight:** Prompts are not the right abstraction. Programs are. The compiler optimizes the prompts.

## Architecture

### Signatures
Declarative specifications of input/output behavior:
```python
class QA(dspy.Signature):
    """Answer questions about the context."""
    context: str = dspy.InputField()
    question: str = dspy.InputField()
    answer: str = dspy.OutputField()
```

### Modules
Composable components that replace hard-coded prompting strategies:
- `dspy.ChainOfThought` — adds reasoning steps
- `dspy.ReAct` — tool-use agent loop
- `dspy.ReActV2` — improved ReAct (new)
- `dspy.ProgramOfThought` — code generation
- Custom modules via `dspy.Module`

### Optimizers (formerly Teleprompts)
Automatically tune prompts and demonstrations:
- `GEPA` — Reflective prompt evolution
- `SIMBA` — For larger LLMs and harder tasks
- `MIPROv2` — Multi-prompt optimization
- `BootstrapFewShot` — Self-generate demonstrations

### Compilation
```python
tp = dspy.GEPA(metric=semantic_f1, auto="medium")
opt = tp.compile(rag, trainset)
# Before: 0.41 F1 → After: 0.63 F1
```

## Results

- 30-45% improvement in factual accuracy
- ~25% reduction in hallucination rates
- Small models (770M T5, Llama2-13b) compiled with DSPy competitive with GPT-3.5 + expert prompts
- Self-bootstrap pipelines outperform expert-created demonstrations by 5-46%

## Multi-Agent Support

DSPy supports agent orchestration:
- `dspy.ReAct` module for tool-use loops
- Multi-agent systems with Genie + DSPy on Databricks
- Agent loops as first-class programs

## Relevance to Pi Extensions

### High Relevance: Optimizable Agent Programs

DSPy's core insight — **programs over prompts, optimize the program** — directly applies to pi workflows. Instead of the model writing a workflow script from scratch, it could write a DSPy-style program that gets compiled/optimized.

### Patterns to Steal

1. **Declarative signatures** — specify what an agent does, not how. Could define agent roles as signatures.
2. **Compilation** — optimize agent prompts against metrics. Relevant for autoresearch's evaluation harness.
3. **Module composition** — agents as composable modules with typed interfaces.
4. **Metric-driven optimization** — define what "good" means, let the compiler find it.

### Limitations for Pi Context

- DSPy is a **framework**, not a tool pattern. Heavy dependency (Python, many transitive deps).
- Compilation requires training data (even 20 examples). Most pi tasks are one-off.
- Optimizer runtime can be minutes. Not suitable for interactive use.
- Overkill for simple subagent delegation. Right for autoresearch-style loops.

### Integration Idea

DSPy as an **autoresearch backend**: define the optimization target as a DSPy metric, use DSPy's optimizer to tune the agent program over iterations. This would give autoresearch principled optimization instead of ad-hoc keep/revert.

## Key Papers

- **DSPy** (Oct 2023) — original framework
- **DSPy Assertions** (Dec 2023) — computational constraints for self-refinement
- **Optimizing Instructions and Demonstrations** (Jun 2024) — multi-stage optimization
- **GEPA** (Jul 2025) — reflective prompt evolution outperforms RL
