# Cross-Domain Execution Patterns

*Generated 2026-03-27 18:00 UTC from reflexion-log.jsonl (114 matched entries)*

These patterns appear across **11** domains. They are generalizations extracted from the reflexion log — patterns that worked repeatedly across different problem classes.

## 1. Targeted Minimal Edits

**Domains:** autonomy-advance:daily, autonomy-improvement:daily, cron, curiosity-engine, proactive-work (5 domains, 24 occurrences)

**Principle:** Make the smallest possible change that fixes the problem. Wide edits introduce regressions.

**Evidence:**
- [curiosity-engine] *curiosity-engine NameError fix* — grep for bare WORKSPACE variable, targeted single-line edit, re-ran engine to confirm cycle complete
- [proactive-work] *Enhanced drift-detector with operation filtering + median-based duration analysi* — Targeted incremental edits + immediate execution validation on real metrics
- [curiosity-engine] *Fixed drift-detector repeated_error duplicate condition bug* — Targeted source inspection, exact-line patch, immediate runtime + compile validation

## 2. Guardian Gate Every Change

**Domains:** autonomy-advance:daily, autonomy-improvement:daily, curiosity-engine, proactive-work (4 domains, 22 occurrences)

**Principle:** Every non-trivial code change must pass guardian verification before being declared complete.

**Evidence:**
- [proactive-work] *added JSON output mode to drift-detector* — minimal argparse migration + preserving existing text output path + immediate guardian validation
- [autonomy-improvement:daily] *subagent-monitor adaptive timeout+retry hardening* — Targeted reliability gap selection from evolution-engine contract plus minimal high-leverage patch w
- [proactive-work] *Added drift-detector --summary-only mode and optimized curiosity-engine performa* — targeted hot-path edits with immediate guardian smoke proof

## 3. Validate Immediately After Change

**Domains:** autonomy-improvement:daily, cron, curiosity-engine, proactive-work (4 domains, 17 occurrences)

**Principle:** Run proof of correctness right after every edit — do not defer validation.

**Evidence:**
- [curiosity-engine] *curiosity-engine NameError fix* — grep for bare WORKSPACE variable, targeted single-line edit, re-ran engine to confirm cycle complete
- [proactive-work] *Fix reliability guard regression (self-healer timeout + drift-detector false pos* — Added timeouts to openclaw calls and removed brittle drift text pattern matching; validated by runni
- [proactive-work] *Enhanced drift-detector with operation filtering + median-based duration analysi* — Targeted incremental edits + immediate execution validation on real metrics

## 4. Gap-Driven Work Selection

**Domains:** autonomy-improvement:daily, cron, curiosity-engine, proactive-work (4 domains, 12 occurrences)

**Principle:** Select work from actual measured gaps (error patterns, missing milestones, capability holes) not intuition.

**Evidence:**
- [proactive-work] *Fix reliability guard regression by correcting metrics integrity verification be* — Targeted source inspection and minimal patch to treat unsealed entries as migration gaps, then valid
- [autonomy-improvement:daily] *subagent-monitor adaptive timeout+retry hardening* — Targeted reliability gap selection from evolution-engine contract plus minimal high-leverage patch w
- [proactive-work] *Built goal-generator.py (self-directed goal creation) + fixed strategic-planning* — Gap signal analysis from changes.md + milestone state + keyword scoring → clean domain ranking; fix 

## 5. Telemetry-First Design

**Domains:** autonomy-advance, cron, curiosity-engine, proactive-work (4 domains, 7 occurrences)

**Principle:** Instrument before optimizing. You can't improve what you can't measure.

**Evidence:**
- [proactive-work] *Add wall-clock timing to performance tracker + fix drift false positives* — Root-cause first: traced 367% claim → found drift detector comparing estimated ms values (minutes×60
- [proactive-work] *Enhanced drift-detector with operation filtering + median-based duration analysi* — Targeted incremental edits + immediate execution validation on real metrics
- [cron] *Build cron-timeout-guard.py (reliability-timeout-guard contract)* — Evolution engine correctly identified real gap: cron jobs had no timeout telemetry. Reused adaptive-

## 6. Deterministic Tests Over Real I/O

**Domains:** autonomy-improvement:daily, cron, proactive-work (3 domains, 7 occurrences)

**Principle:** Use monkeypatching or synthetic fixtures to make tests deterministic. Real I/O in tests = flaky tests.

**Evidence:**
- [cron] *autonomy-advance:daily timeout guard hardening* — Targeted helper abstraction + transient error taxonomy + deterministic monkeypatch test
- [cron] *autonomy-improvement:daily safe_subagent reliability-timeout-guard* — Contract-driven focus + deterministic monkeypatch smoke test enabled fast reliable implementation an
- [cron] *autonomy timeout/retry hardening continuation* — Implementing targeted guardrails directly in hot paths with deterministic monkeypatch tests

## 7. Close the Feedback Loop

**Domains:** autonomy-advance, autonomy-advance:daily, proactive-work (3 domains, 4 occurrences)

**Principle:** Results that don't feed back into next-cycle decisions are wasted. Explicitly wire outputs to inputs.

**Evidence:**
- [autonomy-advance] *speed-hotpath-profiler implementation* — Telemetry-first design + selection-loop integration
- [proactive-work] *Built auto-repair-engine.py for Goal #8 M2: 6 auto-repair strategies for rate_li* — Direct gap analysis → build missing piece → validate → wire cron → close loop
- [proactive-work] *Fix broken curiosity-engine.py syntax* — Targeted syntax fixes via precise line edits + iterative compile validation closed the loop efficien

## 8. Reuse Before Building

**Domains:** autonomy-advance:daily, cron, proactive-work (3 domains, 3 occurrences)

**Principle:** Always check if an existing tool/module solves the problem before writing new code.

**Evidence:**
- [cron] *Build cron-timeout-guard.py (reliability-timeout-guard contract)* — Evolution engine correctly identified real gap: cron jobs had no timeout telemetry. Reused adaptive-
- [proactive-work] *built perf-drift-triage automation* — single-file orchestration reusing existing drift/hotspot/optimizer tools with JSON artifact output
- [autonomy-advance:daily] *autonomy-advance daily: repair-gated self-healer* — Reused failure-triage classifier as single diagnosis source; added cooldown and attempt budget gate

## 9. Implementation-First Learning

**Domains:** curiosity-engine, proactive-work (2 domains, 12 occurrences)

**Principle:** Commit to building the artifact before researching. Forces clarity of intent and yields better retention.

**Evidence:**
- [proactive-work] *Built tools/self-modification.py — safe self-modification with backup/validate/r* — implementation-first: wrote full tool, found rollback bug during tests, fixed it, all 11 tests pass
- [proactive-work] *added checkpoint recommendation scoring* — single-file capability addition with immediate CLI proof and guardian verification
- [proactive-work] *built perf-drift-triage automation* — single-file orchestration reusing existing drift/hotspot/optimizer tools with JSON artifact output

## 10. Cooldown + Retry for Transient Failures

**Domains:** autonomy-advance:daily, cron (2 domains, 4 occurrences)

**Principle:** Rate limits and transient errors need persistent cooldown state, not just retry logic.

**Evidence:**
- [cron] *autonomy-advance:daily timeout guard hardening* — Targeted helper abstraction + transient error taxonomy + deterministic monkeypatch test
- [autonomy-advance:daily] *autonomy-advance adaptive timeout guard* — Integrating timeout policy at safe_subagent layer spreads reliability gains across all callers; retr
- [autonomy-advance:daily] *autonomy-advance daily: repair-gated self-healer* — Reused failure-triage classifier as single diagnosis source; added cooldown and attempt budget gate

## 11. Narrow Scope Before Acting

**Domains:** curiosity-engine, session-2026-02-28 (2 domains, 2 occurrences)

**Principle:** Reduce the blast radius of any scan/search to structured data only. Narrative/text files produce false positives.

**Evidence:**
- [session-2026-02-28] *fixed self-modifier false positives by excluding narrative files from scan* — Narrow scan scope to structured data only — narrative text produces false positives
- [curiosity-engine] *Fixed drift-detector repeated_error duplicate condition bug* — Targeted source inspection, exact-line patch, immediate runtime + compile validation

---

## How to Use These Patterns

1. **Before starting work:** Check which patterns apply to the task at hand.
2. **During implementation:** Apply at least 2–3 patterns per session.
3. **After completion:** Log what worked → reflexion-loop.py → feeds next synthesis.

These patterns compound. Applying them consistently is what separates reliable execution from hit-or-miss.
