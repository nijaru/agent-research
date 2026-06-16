# Context Management in LLM Agents

Research findings on context window management, compaction, and degradation.

## Key findings

### Sculptor (Tsinghua, 2025)

LLMs with context management tools show 27+ point benchmark improvements. Unguided tool use is suboptimal — prompt guidelines are critical for quality.

### Degradation curve

60-70% context fill is where quality starts dropping (Zylos, JetBrains, multiple sources). By 200k tokens many providers hit a price cliff.

### Tool results dominate context

5 file reads = 81% of a 200k window. Conversation history is a small fraction. Context management should focus on tool output, not chat messages.

### LLM-powered compaction

LLM-driven summarization is safe and effective. pi's built-in compaction doesn't bust cache and produces structured summaries (goal, constraints, progress, decisions, next steps).

### Cursor self-summarization

Trained self-summarizers use 5x fewer tokens than prompted ones, with 50% less error. The LLM can learn to summarize well with the right guidance.

### RLM (MIT, 2025)

Alternative approach: treat long prompts as external environment, not direct input. The LLM programmatically inspects and recursively processes snippets. Handles inputs 2 orders of magnitude beyond context limits, outperforms compaction by 26%.

## As of

June 2026.
