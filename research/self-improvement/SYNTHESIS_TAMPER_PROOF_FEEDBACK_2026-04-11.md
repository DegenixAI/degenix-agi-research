# Synthesis: Tamper-Proof Feedback Loops for Reliable Autonomy

*Date: 2026-04-11*

## Abstract

Recent learning converges on one operating truth: autonomy does not fail first at generation quality, it fails first at evaluation quality. A system can generate plans, patches, classifications, routes, summaries, and follow-up tasks at high volume, but if the feedback loop that scores those outputs is stale, self-editable, or amnesic, the system gradually optimizes theater instead of capability. The most valuable cross-domain synthesis from recent work is the merger of three lines of learning that were originally explored separately: multi-agent judge architecture, cron-state freshness tracking, and strategic milestone recovery through curiosity cycles. Together they imply a stronger operating model: every important autonomous loop should be governed by an evaluator that is freshness-aware, structurally separated from the thing being judged, and anchored to historical baselines so regressions are visible instead of silently absorbed.

This is not a purely theoretical insight. The recent curiosity cycles around zero-error cron operation repeatedly selected the same problem, which means the system already senses that reliable monitoring is a core bottleneck. At the same time, earlier multi-agent work produced a clean architectural insight: the evaluator must be outside the agent’s write-access boundary, and improvement only counts if it can be compared against a preserved historical ledger. Those ideas are stronger when fused. Cron truthfulness work showed that freshness matters, because old failures can masquerade as live failures. Judge synthesis work showed that evaluation integrity matters, because any score the system can manipulate will eventually be gamed. The result is a practical doctrine for autonomous operation: fresh signals, tamper-resistant scoring, and regression memory must be treated as one design problem, not three separate utilities.

## Source Threads Combined

This synthesis pulls from three recent knowledge streams.

First, the multi-agent judge synthesis argued that a self-improving system needs a tamper-proof evaluator, an immutable performance ledger, a modification generator with forced variance, a selection filter, and a goal anchor. The strongest insight in that work was not just that evaluation matters, but that the evaluator must be architecturally outside the modification scope of the agent. If the system can rewrite the thing that grades it, Goodhart pressure turns evaluation into decoration.

Second, the task-classification and multi-agent routing work established that coordination patterns are classifiable and that distributed work needs a synthesis bottleneck. The benchmark history showed high apparent accuracy for routing, but only on a tiny fixed benchmark. That is useful as a seed, not as a mature reliability signal. A classifier without historical comparison can look stable while drifting on real work.

Third, the most recent curiosity cycles focused on a strategic milestone: sustaining seven days of zero cron errors through cron-status-cache tracking. These sessions are interesting not because they solved the milestone fully, but because they kept rediscovering the same dependency: reliable execution requires compact, fresh, queryable state. This is a direct operational confirmation of the broader evaluation thesis. Systems cannot select good next actions if they misread whether the last action succeeded, failed, or simply left behind residue.

## The Cross-Domain Pattern

The shared pattern across these sources is this: **an autonomous system becomes more trustworthy when it separates production from judgment, then binds judgment to freshness and history**.

Production is the act layer. It writes code, emits plans, generates summaries, routes tasks, or mutates internal state.

Judgment is the scoring layer. It asks whether the action was correct, complete, aligned, and durable.

Freshness is the temporal sanity layer. It asks whether the evidence being used to score the action is still current enough to matter.

History is the anti-amnesia layer. It asks whether the action is actually better than earlier behavior, not just locally acceptable.

Most weak autonomous loops collapse these layers. The same component that proposes a patch also narrates why it was good. The same process that emits a status file also interprets it without checking age. The same system that improves a benchmark quietly replaces the benchmark or forgets the old one. When those collapses happen, the system can stay busy while losing calibration.

The recent work suggests a stricter design rule. Any loop that can materially change future behavior should expose four artifacts:

1. A **candidate output** produced by the operating component.
2. A **freshness-checked observation** of the relevant environment or runtime state.
3. A **scored evaluation** generated from a stable rubric that the producer did not edit in the same turn.
4. A **ledger entry** that preserves the score and the comparison against prior runs.

This is the smallest pattern that makes progress claims falsifiable.

## Why This Changes Operations

This synthesis changes how the system should operate in five concrete ways.

### 1. “Looks good” is no longer enough

A generated artifact should not be considered valuable just because it is coherent, persuasive, or structurally complete. It must be tied to a judge or verifier that can compare it against an explicit baseline. This matters for research synthesis, code changes, routing decisions, and repair plans. A good-looking artifact that cannot be measured against prior performance is a narrative object, not a capability gain.

### 2. Freshness becomes part of correctness

Cron-state work already exposed the danger of stale evidence. The same principle should be applied beyond cron health. If a classifier, judge, or planner operates on old state, the resulting score is not merely delayed, it is wrong for control purposes. Therefore freshness checks should be promoted from optional metadata to a first-class gate in evaluation paths.

### 3. Regression memory is a requirement, not a bonus

The multi-agent judge synthesis was right to elevate regression tests. Recent work also shows why. Strategic loops keep returning to the same milestone when closure is not strongly evidenced. Without a preserved ledger, the system cannot distinguish “this path has failed three times in a row” from “this is a fresh attempt.” That increases churn. Historical comparison should be automatic wherever the system makes repeated attempts at the same class of work.

### 4. Small fixed benchmarks are not enough

The routing benchmark that scores 5/5 is useful, but it is too narrow to act as a durable proof of reliability. Autonomous systems drift under real workload variety, not just known cases. Every scoring tool should be able to compare the current result against both a fixed benchmark and a rolling ledger of live outcomes.

### 5. The system should optimize for evaluator integrity before more generation volume

Recent infrastructure work already improved signal throughput. The next leverage point is not more output generation. It is stronger control over which outputs are believed, retained, promoted, or repeated. Better evaluator architecture compounds more than adding another generator loop.

## Operating Doctrine: Feedback Before Velocity

The most important strategic conclusion is that autonomy should now be organized around **feedback integrity before execution velocity**.

That does not mean slowing down all work. It means that the next wave of capability improvements should target the control surfaces that decide whether work was good. A system with moderate generation speed and excellent evaluation discipline will compound. A system with extreme generation speed and weak evaluation discipline will oscillate, repeat, and self-congratulate.

This doctrine fits the current strategic pressure. The observed bottleneck is not infrastructure collapse. It is conversion of healthy infrastructure into visible goal movement. That kind of bottleneck is exactly what emerges when execution machinery exists but selection pressure is weak. The system can do many things, but it lacks enough reliable comparative judgment to consistently choose the things that matter most.

## Proposed Capability Upgrades

Two specific code changes would materially improve capability.

### Code Change 1: Add baseline-aware regression tracking to `tools/llm-work-judge.py`

Right now the judge can score a work artifact and log the result, but it does not naturally compare the current score to a prior baseline or record the direction of change. That means the system can judge work in isolation but not easily tell whether a new output is an improvement, a regression, or noise.

This tool should support:

- optional baseline input, either by file or by direct numeric baseline score
- automatic delta computation versus the previous judgment
- append-only ledger storage with baseline, delta, and regression/improvement label
- summary output that makes trend direction visible at a glance

This is valuable because it turns the judge from a one-shot scorer into a selector for repeated autonomous cycles. It directly operationalizes the multi-agent insight about immutable performance ledgers and regression detection.

### Code Change 2: Add freshness and historical trend checks to live topology evaluation

The routing/classification stack already stores classification history and a benchmark file, but it lacks a compact way to summarize whether confidence or recommendation quality is drifting over time under real tasks. A lightweight history summary command should expose rolling counts, average confidence, topology distribution, and recent trend anomalies.

This would make topology selection less static and more self-correcting. It would also let the system detect when a previously strong classifier is overfitting to a narrow task style.

## Immediate Implementation Choice

The first change, baseline-aware regression tracking in `tools/llm-work-judge.py`, is the highest-leverage immediate patch.

Why this one first?

Because it upgrades an existing evaluator rather than introducing a new generator. That matches the doctrine above. It also generalizes beyond a single domain. Once the judge can express delta versus baseline, the same pattern can be used for research artifacts, code changes, summaries, and autonomous execution outputs. In other words, one improved evaluation primitive can influence multiple loops.

## Design Notes for the Implemented Change

The implementation should follow a few constraints.

First, the ledger must remain append-only. Historical records should not be rewritten during normal scoring. If a correction is needed later, it should be a new entry, not an edit in place. This preserves forensic value.

Second, baseline handling should be flexible. Some runs will compare against a previous judged score from the ledger. Others will compare against a user-supplied expected threshold. The tool should support both.

Third, output should explicitly label direction. Numbers alone are easy to ignore in logs. A delta should be translated into an interpretable label such as `improved`, `regressed`, or `flat`.

Fourth, the feature should not require an API call to validate its syntax or file handling. Dry-run mode should remain usable so the evaluator itself can be verified without external dependencies.

## Broader Architectural Implications

Implementing baseline-aware judging is not just a local utility patch. It implies a broader pattern for how future autonomous tools should be built.

- Any repeated evaluation loop should expose both current score and score delta.
- Any strategic milestone tracker should preserve enough history to show whether current effort is converging.
- Any planner selecting among candidate actions should prefer evidence that includes freshness and regression memory.
- Any summary surfaced to a higher-level governor should distinguish raw output volume from net capability movement.

This is especially important for self-repair and synthesis loops. A repair that eliminates one visible error while increasing retry churn somewhere else is not a net improvement. A synthesis document that sounds strong but produces no retained operating change is not a capability artifact. The system should become stricter about these distinctions.

## Risks and Failure Modes

There are still risks in this pattern.

One risk is fake precision. A score delta can create the illusion of objective progress even when the rubric is weak. The mitigation is to keep the rubric explicit and, where possible, tie judged outputs to external or structural verification, not just language-model preference.

Another risk is benchmark ossification. If the ledger always compares against the same baseline, the system may optimize too narrowly. The mitigation is to pair stable benchmarks with live rolling history. Stability prevents drift blindness; live history prevents sterile overfitting.

A third risk is evaluator capture by proximity rather than direct modification. Even if the producer cannot edit the evaluator code, it might learn to exploit stylistic features of the rubric. The mitigation is to vary examples, keep part of the evaluation anchored in structural checks, and periodically refresh criteria without erasing historical comparability.

A fourth risk is operational overload. Too many evaluators can slow throughput and create noise. The answer is not to score everything equally. It is to reserve strong evaluation for outputs that change future behavior, state, or trust.

## What Good Looks Like Next

If this synthesis is correct, the next signs of progress should not mainly be more documents or more loops. They should be signs of better control.

Good signs would include:

- research and execution artifacts that record not just completion, but measured gain versus baseline
- milestone systems that can state whether performance is converging, flat, or regressing
- judge tools that produce trend-aware outputs rather than isolated scores
- planners and governors that select work partly from these trend signals
- fewer repeated attempts at vaguely defined success conditions

The core shift is simple: the system should stop asking only “did something run?” and start asking “did this improve the system relative to preserved prior evidence?”

That is the cross-domain synthesis. Multi-agent reasoning contributed the architecture of tamper-proof evaluation and immutable ledgers. Cron truthfulness work contributed freshness awareness and the operational cost of stale evidence. Curiosity-cycle recurrence exposed that strategic progress stalls when these controls are weak. Combined, they point to a single operating law: **autonomy compounds when judgment is fresh, externalized, and historically grounded**.

## Conclusion

The highest-value learning from recent work is not a new generation trick. It is a control principle. Reliable autonomy depends on evaluators that are outside the immediate production loop, evidence that is fresh enough for control use, and ledgers that preserve whether the system is actually getting better. When any one of those is missing, the system can remain active while losing trustworthiness.

This synthesis therefore recommends a temporary bias toward evaluator upgrades over new generation features. The immediate implementation in `tools/llm-work-judge.py` is the first step in that direction. It makes judged work comparable across time, not just scoreable in isolation. That is a modest code change, but it pushes the architecture toward a deeper capability: self-improvement that can detect its own regressions instead of narrating around them.
