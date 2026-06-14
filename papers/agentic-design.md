# Automated Design of Agentic Systems

## ADAS (ICLR 2025 Outstanding Paper)

**Paper:** "Automated Design of Agentic Systems" — Hu & Clune
**Code:** https://github.com/ShengranHu/ADAS
**Award:** Outstanding Paper, NeurIPS 2024 Open-World Agent Workshop

### Core Idea

**Meta Agent Search**: a meta-agent iteratively programs new agents in code, tests them, adds to archive, uses archive to inform next iteration.

Key claim: agents can be defined in code (Turing Complete), so a meta-agent can theoretically discover **any possible agentic system** — novel prompts, tool use, workflows, and combinations.

### Results

- Agents invented by Meta Agent Search **maintain superior performance across domains and models**
- Progressive discovery of novel agent designs
- Outperforms hand-designed agents on coding, science, and math

### Relevance to Pi

ADAS suggests that **agent design itself can be automated**. Instead of hand-crafting agent definitions, a meta-process could discover better ones. Relevant for autoresearch: the optimization target could be the agent program, not just the task output.

## Agent-as-a-Judge (ICML 2025)

**Paper:** Zhuge et al.
**Code:** https://github.com/metauto-ai/agent-as-a-judge

### Core Idea

Extend LLM-as-a-Judge with **agentic features**: planning, tool-augmented verification, multi-agent collaboration, persistent memory. Enables intermediate feedback for entire task-solving processes.

### Key Finding

Agent-as-a-Judge **dramatically outperforms LLM-as-a-Judge** and matches human evaluation reliability. The step-by-step feedback during intermediate stages is what makes the difference.

### Relevance to Pi

Directly applicable to pi's `verify()` and `judgePanel()` quality patterns. Instead of a single LLM call to judge output, use an agentic judge that can:
- Inspect intermediate steps
- Use tools to verify claims
- Provide structured feedback at each stage

## RAAS (CVPR 2026)

**Paper:** "LLM Agentic System Architecture Search with GRPO" — Yang et al.

### Core Idea

Automates agentic system design with **stable evaluation**:
- **Contextual Architecture Orchestration (CAO)**: evaluate cohorts on same queries, derive merit through peer comparison
- **Multi-Trial Assessment Synthesis (MTAS)**: aggregate performance across multiple trials for statistical robustness

### Results

- HumanEval pass@1: 92.23% → 96.31%
- MATH accuracy: 52.08% → 60.87%

### Relevance to Pi

RAAS's evaluation stability techniques are directly useful for autoresearch:
- **Peer comparison** instead of absolute scores
- **Multi-trial assessment** instead of single runs
- These address the exact problem of "was this iteration actually better?"

## ABSTRAL (2026)

**Paper:** "Automated Multi-Agent System Design via Skill-Referenced Adaptive Search"

### Core Idea

Treats MAS architecture as an evolving **natural-language document** (SKILL.md) refined through contrastive trace analysis.

### Key Findings

1. **Multi-agent coordination tax**: ensembles achieve only 26% turn efficiency, 66% of tasks exhaust turn limit
2. **Design knowledge transfers**: SKILL.md documents learned on one domain provide head start on new domains
3. **Contrastive trace analysis** discovers specialist roles absent from initial design

### Relevance to Pi

ABSTRAL validates the **SKILL.md** approach (already used in pi):
- Natural-language documents as design artifacts
- Transferable design knowledge
- Iterative refinement through trace analysis

The 26% turn efficiency finding is important: **multi-agent systems have significant overhead**. Fan-out only pays when the task is genuinely parallelizable.

## Synthesis: What These Papers Tell Us

| Paper | Key Insight | Pi Application |
|-------|-------------|----------------|
| ADAS | Agent design can be automated | Autoresearch optimization target |
| Agent-as-a-Judge | Agentic evaluation > LLM evaluation | verify(), judgePanel() patterns |
| RAAS | Stable evaluation needs peer comparison + multi-trial | Autoresearch iteration assessment |
| ABSTRAL | Design knowledge in documents, transfers across domains | SKILL.md as optimization target |
| Workflow-R1 | Multi-turn > one-shot for workflows | Script-as-plan pattern validation |

### The Emerging Pattern

The field is converging on: **agents that design and evaluate other agents**, with:
1. Code as the design medium (ADAS)
2. Natural language as the design document (ABSTRAL)
3. Agentic evaluation with intermediate feedback (Agent-as-a-Judge)
4. Stable multi-trial assessment (RAAS)
5. RL at the sub-sequence level (Workflow-R1)

This is exactly what pi's architecture enables: model writes code (workflows), documents design (SKILL.md), evaluates results (verify/judgePanel), and iterates (autoresearch).
