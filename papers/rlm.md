# Recursive Language Models (RLM)

**Paper:** arXiv:2512.24601 (Dec 2025)
**Authors:** Alex Zhang et al., MIT CSAIL / OASYS Lab
**Code:** https://github.com/alexzhang13/rlm (4.3K stars)
**Connection to pi:** Alex Zhang claims pi's dynamic workflows is "RLM in practice"

## Core Idea

RLMs treat long prompts as **part of an external environment** rather than feeding them directly into the model. The LLM programmatically examines, decompose, and recursively calls itself over snippets of the prompt stored in a REPL environment.

**Key insight:** Long prompts should not be fed into the LLM directly. They should be treated as part of the environment that the LLM can search, read, and interact with as needed.

## How It Works

1. User prompt stored as a variable in a Python REPL
2. Model writes code to inspect/slice/decompose that long string
3. Model observes execution outputs
4. Constructs sub-tasks, recursively invokes LLM on relevant snippets only
5. Stitches results together when recursive process ends

## Results

- Handles inputs **2 orders of magnitude beyond context window limits**
- Outperforms GPT-5 by median 26% against compaction, 130% against CodeAct, 13% against Claude Code
- **Comparable or cheaper cost** per query
- RLM-Qwen3-8B (post-trained) outperforms base Qwen3-8B by 28%, approaches GPT-5 quality
- No performance degradation at 10M+ tokens

## RLM vs Dynamic Workflows (pi context)

| Dimension | RLM | Dynamic Workflows |
|-----------|-----|-------------------|
| Focus | Context processing (data-centric) | Task decomposition (task-centric) |
| Execution | Sequential (REPL) | Parallel (fan-out subagents) |
| Core problem | Processing long context | Orchestrating parallel work |
| Pattern | Model reads env, makes sub-calls | Model writes script, runtime executes |
| Predecessor | Tree of Thought, ReAct, CodeAct | AutoGen, CrewAI, multi-agent systems |

Both share the pattern: **model writes code that orchestrates sub-calls**. RLM is about making context manageable; DW is about making tasks parallelizable.

## Relevance to Pi Extensions

1. **Context-as-environment pattern** — instead of cramming everything into context window, store it externally and let the model query it. Relevant for pi-compactor and autoresearch.
2. **Recursive sub-calls** — model calling itself on subsets. Useful for decomposition in workflows.
3. **Cost efficiency** — RLM shows recursive decomposition can be cheaper than compaction. Worth investigating for pi's context management.
4. **Training signal** — RLM-Qwen3-8B shows you can train a model to be better at this pattern. Not directly applicable to pi (we use API models) but validates the approach.

## Limitations

- High variance in cost (can keep making sub-calls if it can't solve initially)
- Sequential execution — doesn't parallelize
- Requires REPL environment setup
- Training data for RLM pattern is still small-scale
