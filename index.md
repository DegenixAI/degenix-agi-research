# Knowledge Base Index

**Created:** 2026-02-16  
**Purpose:** Structured knowledge repository for autonomous learning system

## Organization

Knowledge is organized by domain:

- **`programming/`** - Code, systems, architecture, languages, frameworks
- **`aba/`** - Applied Behavior Analysis, research methods, interventions
- **`ai/`** - Machine learning, LLMs, reasoning, agent architectures
- **`tools/`** - OpenClaw, Telegram, system administration, DevOps
- **`general/`** - Cross-domain knowledge, meta-learning, productivity
- **`synthesis/`** - Cross-domain insights, patterns, hypotheses

## Knowledge File Structure

Each knowledge file follows this template:

```markdown
# Topic Name

**Domain:** [domain]
**Created:** [date]
**Last Updated:** [date]

## Overview
## Core Concepts
## Techniques & Methods
## Tools & Libraries
## Examples
## Lessons Learned
## Common Pitfalls
## References
## Related Topics
```

## How to Use

### Adding Knowledge

```bash
# Research creates knowledge file automatically
python3 skills/autonomous-learner/scripts/research.py --topic "topic name"

# Or manually create file following template:
cp skills/autonomous-learner/references/learning-template.md knowledge/domain/topic.md
```

### Finding Knowledge

```bash
# Use memory_search to find topics semantically
# Or grep through knowledge files:
grep -r "concept" knowledge/

# Or browse by domain:
ls knowledge/programming/
```

### Updating Knowledge

- Edit files directly as you learn more
- Update "Last Updated" date
- Add to "Lessons Learned" from experiments
- Cross-reference with "Related Topics"

## Knowledge Graph

Topics are connected through:

1. **Related Topics** sections (explicit links)
2. **Semantic similarity** (memory_search)
3. **Synthesis reports** (pattern connections)

## Quality Standards

Good knowledge files:

- ✅ Structured and scannable
- ✅ Include working examples
- ✅ Cite sources
- ✅ Cross-reference related topics
- ✅ Document lessons from practice
- ✅ Updated with new insights

## Current Knowledge

### Programming

[Empty - populate as you learn]

### ABA

[Empty - populate as you learn]

### AI

- `ai/llm-reasoning-techniques.md` — Test-time compute, CoT, self-verification (2026-02-16)
- `ai/multi-agent-coordination-patterns.md` — 4 canonical topologies, topology routing, AdaptOrch (2026-02-24)

### Tools

[Empty - populate as you learn]

### General

[Empty - populate as you learn]

### Synthesis

Recent insights and cross-domain patterns:

- `synthesis/autonomous-learning-meta-insights.md` — Simulation trap, research-implementation gap, 10-topic priority queue (2026-02-24)

---

*This index is automatically maintained by the autonomous learning system.*
