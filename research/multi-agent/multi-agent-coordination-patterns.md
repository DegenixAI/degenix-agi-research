# Multi-Agent Coordination Patterns

**Domain:** ai/agents  
**Created:** 2026-02-24  
**Last Updated:** 2026-02-24  
**Source:** AdaptOrch paper (arXiv:2602.16873), practical experience

## Key Insight: Orchestration > Model Selection

As of early 2026, frontier LLMs (GPT-4o, Claude 3.5/4, Gemini 2.0, Llama 3.3, DeepSeek-V3) cluster within 2–5% of each other on standard benchmarks. This means:

> **Orchestration topology now dominates system performance more than which model you use.**

When models converge in capability, *how you compose them* becomes the primary optimization lever.

## The 4 Canonical Topologies

### 1. Parallel Topology
**When:** Subtasks are fully independent (no data dependencies)
**Pattern:** Spawn N agents simultaneously, each in own context window
**Benefit:** Compresses multi-hour work into minutes
**OpenClaw use:** Research subtopics, code module review, testing multiple approaches

```
Task → [Agent A | Agent B | Agent C] → Synthesis Agent → Output
```

### 2. Sequential Topology  
**When:** Each step depends on the previous output
**Pattern:** Chain of agents where output feeds next input
**Benefit:** Quality validation at each stage
**OpenClaw use:** Draft → Review → Refine → Final

```
Task → Agent A → Agent B → Agent C → Output
```

### 3. Hierarchical Topology
**When:** Complex tasks with partial dependencies
**Pattern:** Orchestrator decomposes into subtasks, specialist sub-agents execute
**Benefit:** Best for projects with mixed serial/parallel work
**OpenClaw use:** Large feature builds, multi-domain research

```
                Orchestrator
               /      |      \
         Agent A   Agent B   Agent C
              \       |       /
               Synthesis Layer
```

### 4. Hybrid Topology
**When:** Complex workflows with multiple interdependency patterns
**Pattern:** Mix of parallel and sequential stages
**Benefit:** Maximizes both speed and quality
**OpenClaw use:** Most sophisticated real-world tasks

## Topology Selection Algorithm (from AdaptOrch)

1. **Decompose the task** into a DAG (directed acyclic graph) of subtasks
2. **Classify edges:**
   - No edges between subtasks → Parallel
   - Linear chain → Sequential  
   - Fan-out then fan-in → Hierarchical
   - Mixed → Hybrid
3. **Route to topology** based on DAG structure

**Key metrics:**
- *Parallelism width:* Max independent subtasks at any level
- *Critical path depth:* Longest dependency chain
- *Inter-subtask coupling:* How much outputs share state

## Synthesis Protocol

When combining parallel agent outputs:
1. **Consistency scoring:** Embed each output, compute similarity matrix
2. **Conflict detection:** Flag outputs with cosine similarity < 0.6
3. **Adaptive re-routing:** For conflicts, spawn resolution agent
4. **Provable termination:** Max 3 resolution rounds before escalation

## Performance Data

AdaptOrch validation results (vs. single static topology):
- **Coding tasks (SWE-bench):** +12% improvement
- **Reasoning tasks (GPQA):** +18% improvement  
- **RAG tasks:** +23% improvement

*Same underlying models, just better orchestration.*

## Practical Application for This System

### Current State
- OpenClaw `sessions_spawn` supports parallel agent execution
- `subagents` tool manages spawned agents
- No automatic topology selection yet

### Gap: Topology-Aware Dispatch
Current approach: Always spawn same way, manual task decomposition
Better approach: Analyze task structure → select topology → dispatch accordingly

### Quick Pattern Library

**Research tasks** → Parallel (subtopics are independent)
```python
# Spawn 3 research agents in parallel
topics = ["implementation", "best practices", "pitfalls"]
for topic in topics:
    sessions_spawn(task=f"Research {topic} of {main_topic}")
```

**Code review** → Parallel (files are independent)  
**Bug fix** → Sequential (understand → diagnose → fix → test)  
**Feature build** → Hierarchical (design → parallel impl → integration)  
**Analysis report** → Hybrid (gather data in parallel, sequential analysis)

## Anti-Patterns to Avoid

1. **Sequential when parallel is possible** — biggest waste of time
2. **Overparallelizing dependent tasks** — causes inconsistent outputs
3. **No synthesis layer** — parallel results without integration = chaos
4. **Context window overflow** — each agent needs independent context
5. **Self-MoA trap** — querying same model multiple times doesn't help as much as diversity

## Integration with OpenClaw

The AGENTS.md rule "spawn to coder if >15 min and 5+ files" is a rough proxy for hierarchical topology. A more principled approach:

1. Estimate parallelism width of the task
2. If width ≥ 3 independent subtasks → spawn parallel agents
3. If depth ≥ 4 sequential steps → consider spawning coordinator
4. Otherwise → handle in main session

## References

- AdaptOrch: Task-Adaptive Multi-Agent Orchestration (arXiv:2602.16873, Feb 2026)
- Claude Code Agent Teams (Anthropic, 2026)
- Multi-Agent Collaboration Mechanisms Survey (arXiv:2501.06322, Jan 2025)
