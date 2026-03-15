# LLM Reasoning Techniques

**Domain:** ai  
**Created:** 2026-02-16  
**Last Updated:** 2026-02-16

## Overview

LLM reasoning techniques transform models from simple text generators into sophisticated problem-solving systems. The key paradigm shift: **allocating test-time compute** - giving models time to "think" before responding, rather than generating immediate outputs.

Recent breakthroughs (2024-2025) moved from fluent one-shot LLMs to **reasoning models** that break problems into steps, explore multiple paths, and self-verify before answering.

## Core Concepts

### Test-Time Compute

**Definition:** Computational resources allocated during inference (not training) to improve reasoning quality. Instead of instant responses, models "think" for seconds, minutes, or longer.

**Key insight:** "AI can be more than chatbots. Inference costs will be higher, but what cost would you pay for a new cancer drug?" - Dr. Noam Brown, OpenAI

**Trade-off:** Higher inference cost ↔ Dramatically better reasoning on complex problems

### Reasoning vs. Generation

- **Traditional LLMs**: Pattern matching → immediate output
- **Reasoning models**: Problem decomposition → iterative thinking → verified output

## Techniques & Methods

### 1. Chain-of-Thought (CoT) Prompting

**Core idea:** Explicitly encourage models to generate intermediate reasoning steps before final answers.

**How it works:**
- Break problems into sequential logical steps
- Verbalize reasoning process
- Maintain context across steps
- Ensure logical consistency

**Example:**
```
Prompt: "Let's solve this step by step..."
Model generates:
Step 1: Identify the problem type
Step 2: Break into sub-problems
Step 3: Solve each sub-problem
Step 4: Combine for final answer
```

**Benefits:**
- Significant performance improvement on complex tasks
- Transparent reasoning (can inspect steps)
- Better error detection

**Variants:**
- **Zero-Shot CoT**: Just add "Let's think step by step"
- **Few-Shot CoT**: Provide examples of step-by-step reasoning
- **Automatic CoT**: Model generates examples itself

**Source:** Wei et al. (2022), foundational work spawning rich ecosystem of techniques

### 2. Self-Consistency

**Core idea:** Generate multiple reasoning paths for the same problem, then select the most consistent answer.

**How it works:**
1. Sample N different reasoning chains (e.g., 5-10)
2. Each chain reaches a conclusion
3. Take majority vote or most consistent output
4. Aggregate to final answer

**Benefits:**
- ~3-5% accuracy improvement over vanilla CoT
- Reduces random errors and hallucinations
- Works well with chain-of-thought

**Challenges:**
- Higher computational cost (multiple samples)
- Requires careful aggregation strategy

**Optimization:** CoT-PoT ensembling - using just 2 diverse strategies can achieve similar results to many samples

**Source:** Wang et al. (2022), arxiv.org/html/2602.11361

### 3. Tree of Thoughts (ToT)

**Core idea:** Enable branching into multiple reasoning paths, evaluate partial results, pursue most promising directions.

**How it works:**
```
Problem
├── Approach A
│   ├── Sub-solution A1 ✓ (promising)
│   └── Sub-solution A2 ✗ (dead end)
├── Approach B
│   └── Sub-solution B1 ✓ (very promising)
└── Approach C
    └── Sub-solution C1 ? (uncertain)
```

**Key features:**
- **Exploration**: Try multiple paths
- **Evaluation**: Score partial solutions
- **Pruning**: Abandon unproductive paths
- **Backtracking**: Return to promising branches

**Benefits:**
- Handles complex, multi-step problems
- Prevents getting stuck in local optima
- More thorough exploration

**Trade-offs:**
- Significantly higher compute cost
- Requires good evaluation heuristics

**Source:** Yao et al. (2023)

### 4. Extended Thinking / Test-Time Scaling (o1-style)

**Core idea:** Let models "think" for extended periods (seconds → hours → days) using reinforcement learning-trained reasoning strategies.

**How o1 works:**
1. **Hidden chain-of-thought**: Internal reasoning not shown to user
2. **RL-optimized reasoning**: Trained via reinforcement learning for sophisticated problem-solving
3. **Dynamic compute allocation**: More thinking time for harder problems
4. **Self-verification**: Review and revise before answering

**Key findings:**
- Performance improves with more train-time RL compute
- Performance improves with more test-time thinking
- Emergent complex reasoning strategies from end-to-end RL

**Breakthrough:** OpenAI o1 (Sept 2024) introduced reasoning models that "think before they speak"

**Evolution:**
- **o1-preview** (2024): Think for seconds
- **Future vision**: Think for hours, days, weeks
- **o3** (2025): Complex test-time strategies emerge naturally from RL

**Challenges:**
- **Model underthinking**: Sometimes reaches correct solution but deviates during extended reasoning
- Need for careful RL training to avoid degradation

**Sources:**
- OpenAI o1 paper (2024)
- Wikipedia: Reasoning model
- arxiv.org/html/2502.12215v1

### 5. Reflection / Self-Critique (Reflexion)

**Core idea:** Model evaluates its own outputs, identifies flaws, and iteratively improves.

**How it works:**
1. Generate initial response
2. **Reflect**: "Was this correct? What could be better?"
3. Identify errors or weaknesses
4. **Retry**: Generate improved response
5. Repeat until satisfactory

**Techniques:**
- **Output reevaluation**: Feed output back with prompt "Was the previous answer correct?"
- **Verbal reinforcement**: Articulate what went wrong
- **Multi-agent reflection**: Different agents critique each other
- **Self-awareness scoring**: Evaluate depth of introspection

**Benefits:**
- Internalized self-correction skill
- Fixes contradictions, overconfidence, quality variation
- Critical for AI agents and high-stakes domains

**Applications:**
- Academic writing (response to reviewers)
- Code debugging
- Multi-step planning
- Qualitative analysis

**Advanced:** ReflCtrl - Representation engineering to identify and steer "reflection direction" in LLM activations

**Sources:**
- Shinn et al. (2023): Reflexion paper
- nature.com/articles/s44387-025-00045-3
- galileo.ai/blog/self-reflection-in-language-models

### 6. Hybrid & Advanced Methods

**Retrieval-Augmented Thought (RAT):**
- Merges RAG retrieval into chain-of-thought process
- Iteratively refine and correct reasoning with factual lookups

**Retrieval-Augmented Thought Tree (RATT):**
- Tree-structured reasoning where each node invokes RAG
- Enhances logical soundness and factual reliability

**Least-to-Most Prompting:**
- Start with simplest sub-problems
- Build up to complex problem
- Progressive complexity scaling

**Budget Forcing (s1-32B approach):**
- Dynamic reasoning optimization at test-time
- Achieved o1-level performance with 1,000 training examples
- Lightweight alternative to massive RL pipelines

**Matrix of Thought:**
- Beyond linear chains or trees
- Multi-dimensional reasoning representations

## Tools & Libraries

**Prompting frameworks:**
- LangChain (CoT, ReAct patterns)
- Guidance (constrained generation)
- DSPy (declarative self-improving prompts)

**Reasoning models:**
- OpenAI o1, o3 (proprietary, API)
- DeepSeek R1 (open weights)
- Alibaba QwQ (open reasoning model)

**Evaluation:**
- MATH benchmark (mathematical reasoning)
- GSM8K (grade school math)
- HumanEval (code generation)
- MMLU (multi-domain knowledge)

## Examples

### Basic Chain-of-Thought

**Prompt:**
```
Question: If a train travels 120 miles in 2 hours, how far will it travel in 5 hours at the same speed?

Let's solve this step by step:
```

**Response:**
```
Step 1: Calculate the train's speed
- Distance = 120 miles
- Time = 2 hours
- Speed = 120 ÷ 2 = 60 mph

Step 2: Calculate distance for 5 hours
- Speed = 60 mph
- Time = 5 hours
- Distance = 60 × 5 = 300 miles

Answer: The train will travel 300 miles in 5 hours.
```

### Self-Consistency Example

Generate 3 reasoning paths:
1. Path A: Speed → Distance (concludes 300 miles)
2. Path B: Ratio approach → 120/2 = x/5 (concludes 300 miles)
3. Path C: Unit conversion (concludes 300 miles)

**Majority vote:** 300 miles (3/3 consensus) ✓

### Reflection Example

**Initial response:**
```
The capital of France is London.
```

**Reflection prompt:**
```
Review the previous answer. Is it correct? What's wrong?
```

**Self-critique:**
```
No, that's incorrect. London is the capital of the United Kingdom, not France. 
The correct answer is Paris.
```

**Corrected response:**
```
The capital of France is Paris.
```

## Lessons Learned

### What Works

- **CoT dramatically improves complex reasoning** - simple "step by step" prompt can boost accuracy 20-40%
- **Self-consistency trades compute for accuracy** - effective for high-stakes tasks
- **Extended thinking scales** - more test-time compute → better results (up to a point)
- **Reflection catches errors** - iterative self-critique improves quality
- **Combining techniques compounds benefits** - CoT + Self-Consistency + Reflection > any single method

### What Doesn't

- **More steps ≠ always better** - underthinking phenomenon: models can reach correct solution then deviate
- **Reflection without grounding fails** - self-critique needs external feedback or verification
- **Pure prompt engineering has limits** - RL-trained reasoning (o1-style) outperforms prompting alone
- **One-size-fits-all doesn't work** - different tasks need different techniques

### How to Choose

| Task Type | Recommended Technique |
|-----------|---------------------|
| Simple Q&A | Standard prompting |
| Math/logic | Chain-of-Thought |
| Critical decisions | Self-Consistency |
| Open-ended exploration | Tree of Thoughts |
| Long-form reasoning | Extended thinking (o1) |
| Error-prone tasks | Reflection |
| Factual + reasoning | RAT (retrieval + CoT) |

## Common Pitfalls

### Pitfall 1: Assuming More Tokens = Better Reasoning

**Problem:** Longer responses don't always mean better reasoning. Models can verbosely arrive at wrong answers.

**Solution:** Evaluate reasoning quality, not length. Use self-consistency to catch verbose errors.

### Pitfall 2: No Verification Loop

**Problem:** Chain-of-thought can propagate errors through steps.

**Solution:** Add self-verification: "Is this step correct? Let me check..."

### Pitfall 3: Ignoring Compute Costs

**Problem:** Tree-of-thought with high branching factor becomes prohibitively expensive.

**Solution:** Strategic pruning, budget forcing, or hybrid approaches (CoT for most, ToT for critical branches)

### Pitfall 4: Overfitting to Prompts

**Problem:** Carefully crafted few-shot examples don't generalize.

**Solution:** Zero-shot CoT or train reasoning with RL (o1 approach)

### Pitfall 5: Reflection Without Grounding

**Problem:** Model critiques itself in circles without external truth.

**Solution:** Reflection + tool use (calculators, code execution, web search for facts)

## References

- [Wei et al. (2022): Chain-of-Thought Prompting](https://arxiv.org/abs/2201.11903) - Foundational CoT paper
- [Wang et al. (2022): Self-Consistency Improves Chain of Thought Reasoning](https://arxiv.org/abs/2203.11171)
- [Yao et al. (2023): Tree of Thoughts](https://arxiv.org/abs/2305.10601)
- [OpenAI o1 System Card (2024)](https://openai.com/index/learning-to-reason-with-llms/) - Test-time compute reasoning
- [Shinn et al. (2023): Reflexion](https://arxiv.org/abs/2303.11366) - Self-reflection for agents
- [IBM: What is Chain of Thought Prompting?](https://www.ibm.com/think/topics/chain-of-thoughts) (Nov 2025)
- [AltexSoft: Chain-of-Thought Prompting Explained](https://www.altexsoft.com/blog/chain-of-thought-prompting/) (Sept 2025)
- [Galileo: 8 Chain-of-Thought Techniques](https://galileo.ai/blog/chain-of-thought-prompting-techniques) (Aug 2025)
- [Nature: Self-reflection enhances LLMs](https://www.nature.com/articles/s44387-025-00045-3) (Dec 2025)
- [arXiv: Revisiting Test-Time Scaling of o1-like Models](https://arxiv.org/html/2502.12215v1) (Feb 2025)
- [Preprints.org: Reasoning in LLMs - Chain-of-Thought to Massively Decomposed Agentic Processes](https://www.preprints.org/manuscript/202512.2242) (Dec 2025)

## Related Topics

- `agent-architectures.md` - How reasoning integrates into agentic systems
- `prompt-engineering.md` - Broader prompting strategies
- `../programming/python-async.md` - Analogous: event-driven reasoning patterns
- `reinforcement-learning.md` - RL training for reasoning (o1 approach)

---

*Research depth: deep*  
*Status: validated* (through research synthesis, not yet experimentally)

## Next Steps

1. **Experiment**: Implement simple CoT vs Self-Consistency comparison on math problems
2. **Apply**: Use extended thinking mode more deliberately in my own reasoning
3. **Synthesize**: Connect reasoning patterns to ABA behavior analysis frameworks (antecedent → behavior → consequence ~ step-by-step reasoning)
4. **Track**: Monitor when different techniques work best in my daily work
