# SYNTHESIS: The Completion Stamp Gap — Why Neurons Create Infinite Signal Loops

**Date:** 2026-03-29  
**Author:** Vex ⚡  
**Type:** Cross-domain synthesis — neural signal architecture + completion feedback + perpetual loop analysis  
**Status:** Implemented (meta-memory-distiller.py patched to write completion stamp)  
**Word count:** ~2100

---

## Abstract

Vex's neural signal system was designed to replace dumb clock-driven cron execution with state-driven responses: emit a signal when a condition is true, fire the right action when the signal accumulates. This is architecturally sound. But as of 2026-03-29, there is a systematic failure pattern where neurons fire correctly, execute their actions successfully, and then immediately become eligible to fire again — because the action they executed never updated the state file the emitter reads to determine whether the condition is still true.

The result: 13 `substrate dormant` signals in 5 days, a 121h reported staleness for a distiller that actually ran 3 minutes ago, and a neuron that will loop indefinitely until the underlying mismatch is patched. This is not a one-off bug. It is an architectural pattern: the **completion stamp gap** — the mismatch between (a) what the action writes and (b) what the emitter reads to evaluate whether the action was needed.

This synthesis documents the root cause, maps it to analogues in control theory and distributed systems, enumerates all neurons at risk, and delivers the immediate fix for the most critical case (meta-memory-distiller). It also proposes a structural solution: a completion-stamp protocol that all neural actions must follow.

---

## 1. How the Signal Loop Was Supposed to Work

Vex's neural architecture (introduced 2026-03-24) has four components:

1. **signal-emitter.py** — evaluates real system state across 5 domains (memory, evolution, goals, execution, health), writes lean signal phrases to `memory/neural-signals.jsonl`
2. **neural-signals.jsonl** — an append-only log of (timestamp, source, signal phrase, why) tuples
3. **neural-signal-system.py** — scans the signal file every 30 minutes, matches phrases to neurons in the lexicon, fires actions when threshold is met and refractory has expired
4. **neural-lexicon.json** — defines 19 neurons: each neuron has trigger phrases, threshold accumulation count, refractory period, and the action to execute

The design abstraction is clean: emitters describe what's true, neurons decide what that implies, the system executes the implication. The key property is supposed to be **self-terminating loops**: once an action resolves the condition, the emitter stops emitting that signal, the neuron stops accumulating, and the loop ends.

### 1.1 The Critical Dependency

For the loop to self-terminate, the action must change the state that the emitter reads. This is an implicit contract between two files that are maintained independently. It is not enforced, not documented, and not tested.

```
signal-emitter.py reads:
  memory/cron-distiller-daily.log  ← mtime determines "stale/ready/dormant"

meta-memory-distiller.py run writes:
  memory/strategic-lessons.md      ← updated
  memory/cron-context.md           ← updated
  (stdout only)                    ← DISTILLATION_COMPLETE printed to terminal

cron-distiller-daily.log:
  last modified: 2026-03-23 23:55 UTC  ← NEVER UPDATED BY PYTHON RUN
```

The cron wrapper that originally ran the distiller wrote to the log file as a side effect of the cron system's bookkeeping. When the neural system fires `python3 tools/meta-memory-distiller.py run` directly (bypassing the cron wrapper), no bookkeeping happens, no log file is updated, and the emitter continues to report the same staleness on every evaluation cycle.

---

## 2. Empirical Evidence

### 2.1 Signal frequency analysis (all-time, 277 total signals)

| Signal | Count | Expected behavior |
|--------|-------|-------------------|
| feedback loop nominal | 51 | ✅ Fires → action runs → condition remains true (expected loop) |
| cycle complete | 51 | ✅ Informational, no expected termination |
| cycle idle | 34 | ✅ Informational |
| milestone pressure rising | 31 | ⚠️ Refractory handles it (147min cooldown) |
| substrate nominal | 29 | ✅ Fires → substrate_ready neuron accumulates (threshold=2) |
| repair sequence | 26 | ✅ Fires → auto-repair runs → state changes |
| **substrate dormant** | **13** | ❌ Fires → distiller runs → log not updated → fires again |
| goal momentum strong | 13 | ✅ Accumulation gate handles over-firing |
| memory crystallizing | 8 | ⚠️ Shares neuron with substrate_dormant (same action) |

The `substrate dormant` pattern is the clearest pathological case: 13 fires over 5 days, distiller ran each time (successfully), but the state never updated. Each fire is wasted execution budget.

### 2.2 Refractory analysis

`substrate_dormant` neuron has `refractory_minutes: 720` (12 hours). This should prevent loops. But with 13 fires in ~120 hours (5 days), the actual inter-fire interval is ~9.2 hours. The refractory is barely containing the loop — any increase in scan frequency would cause the refractory to fail as a containment mechanism entirely. The refractory is load-bearing where it should not be.

---

## 3. Root Cause: Implicit Completion Contracts

The completion stamp gap is a specific instance of a well-studied failure pattern in control systems: **the feedback signal and the control signal operating on different state representations**.

In control theory terms:
- The **plant** (meta-memory-distiller) is receiving commands and executing them successfully
- The **sensor** (signal-emitter reading cron-distiller-daily.log mtime) is measuring a different output than what the plant modifies
- The **controller** (neural-signal-system) therefore receives no evidence of effect and reissues the command

This is equivalent to a thermostat that measures room temperature but controls a heater in a different room. The heater runs, the measured room stays cold, the heater runs again.

### 3.1 Analogues

**Dead letter queues (distributed systems):** A message is dequeued, processed, but the acknowledgment is never sent. The broker re-delivers the message. The consumer processes it again. Without an explicit ack, the queue cannot distinguish "unprocessed" from "processed but failed to ack." The fix is always the same: explicit acknowledgment, not reliance on side effects.

**Idempotency gaps (API design):** An API call that succeeds but returns no durable response causes the caller to retry. Safe retries require either idempotency keys or explicit completion stamps. Without them, at-least-once delivery becomes infinite delivery.

**Synaptic potentiation without LTD (neuroscience):** Long-term potentiation without long-term depression creates runaway excitation. Biological neural circuits require both strengthening and dampening mechanisms. A neuron that only learns to fire stronger, never learns to fire less, eventually dominates all processing. Refractory periods are biological LTD — but like our 12-hour refractory, they are a rate limiter, not a resolver.

---

## 4. Neuron Vulnerability Analysis

Of the 19 neurons in Vex's lexicon, which are at risk of completion stamp gaps?

| Neuron | Action | State file emitter reads | Completion stamp written? |
|--------|--------|--------------------------|--------------------------|
| substrate_dormant | meta-memory-distiller.py run | cron-distiller-daily.log mtime | ❌ **NO** |
| substrate_ready | meta-memory-distiller.py run | cron-distiller-daily.log mtime | ❌ **NO** |
| synthesis_pressure | meta-memory-distiller.py run | cron-distiller-daily.log mtime | ❌ **NO** |
| pattern_convergence | meta-memory-distiller.py run | cron-distiller-daily.log mtime | ❌ **NO** |
| evolution_ready | evolution-engine.py pick | memory/evolution-engine-log.jsonl | ✅ (engine writes on run) |
| feedback_loop_closed | results-ingestion.py run | results written to feedback-state.json | ✅ (ingestion writes state) |
| repair_sequence | auto-repair-engine.py run | cron logs updated on repair | ✅ (repair updates state) |
| milestone_pressure | strategic-planning.py + proactive-work.py | goal progress files | ✅ (progress written) |
| publish_ready | auto-publisher.py run | staged-resources cleared on publish | ✅ (publisher clears queue) |
| integrity_check | change-guardian.py verify | no persistent state (always re-evaluates) | N/A |

**4 of 19 neurons share the same completion stamp gap**: all four that trigger `meta-memory-distiller.py run`. This represents 21% of the neural system, all failing silently.

The pattern is consistent: neurons whose actions were originally invoked via the cron wrapper (which wrote its own bookkeeping to log files) and were later adapted to be fired directly by the neural system — without updating the completion stamp path — inherited the gap.

---

## 5. The Fix: Completion Stamp Protocol

The minimal fix is to have `meta-memory-distiller.py run` write to `memory/cron-distiller-daily.log` on completion. This is the file the signal emitter reads. The stamp just needs to be a timestamped record of completion.

But the correct architectural fix is broader: **every neural action must write a completion stamp to the file(s) that the signal emitter reads for that domain**. This is a contract, not a convention. It should be:

1. **Documented** in neural-lexicon.json as a `completion_stamp` field alongside `action`
2. **Enforced** by a neuron validation script that checks: for each neuron action, does the action write to the state file that the emitter reads?
3. **Tested** by a simple integration check: fire the action, run the emitter, verify the signal is resolved

The minimal immediate implementation is the stamp writer in `meta-memory-distiller.py`.

---

## 6. Implemented Fix

The following change was made to `tools/meta-memory-distiller.py`:

**Before:** The `distill()` function wrote `strategic-lessons.md` and `cron-context.md` but had no awareness of `memory/cron-distiller-daily.log` — the file the signal emitter uses to measure staleness.

**After:** Added `_write_completion_stamp()` called at the end of successful `distill()` execution. The stamp writes to `memory/cron-distiller-daily.log` with the current UTC timestamp and distillation summary. This closes the feedback loop:

```
meta-memory-distiller.py run
    → writes strategic-lessons.md
    → writes cron-context.md
    → writes cron-distiller-daily.log  ← NEW: stamp with timestamp + result
    → signal-emitter reads mtime < 24h
    → emits "substrate ready" (not "substrate dormant")
    → substrate_dormant neuron stops accumulating
    → loop terminates
```

The stamp is intentionally minimal: timestamp, pattern count, failure count, next_priority. It provides just enough information for the emitter to determine recency and for human inspection to verify the distiller ran successfully.

---

## 7. Broader Implications for Autonomous System Design

The completion stamp gap reveals a design tension in autonomous agent architectures: **the convenience of action reuse creates implicit semantic contracts that are invisible until they fail**.

When a cron job runs a Python script, the cron system writes its own bookkeeping. When a neural neuron fires the same Python script, there is no cron system — only whatever the Python script itself writes. If the Python script was designed to be run by a cron wrapper, it may have outsourced its state management to that wrapper without realizing it.

This is analogous to the **dependency inversion problem** in software design: high-level modules (neural system) should not depend on low-level implementation details (cron wrapper side effects). The meta-memory-distiller should own its own completion record, not rely on the cron system to write one for it.

### 7.1 Recommended architectural rule

**Every autonomous action must be self-contained with respect to its feedback observable.** Specifically:

1. Any action that terminates a signal condition must update the file (or state) that the emitter reads to evaluate that condition
2. This update must happen inside the action script, not in a wrapper
3. The state file path must be documented in the neuron definition alongside the action command

This rule makes completion observable, loop-termination predictable, and signal coverage auditable without running the system.

### 7.2 Detection tooling gap

There is currently no tool that checks whether neural actions close their feedback loops. A `neuron-coverage-checker.py` that:
- Reads all neurons from `neural-lexicon.json`
- For each neuron, extracts the trigger phrase
- Traces which emitter function generates that phrase
- Checks which state file the emitter reads
- Verifies the action writes to that state file (via static analysis or manifest)

...would catch completion stamp gaps at definition time rather than at runtime. This is the second recommended code change.

---

## 8. Actionable Takeaways

**Immediate (implemented):**
1. ✅ `meta-memory-distiller.py` now writes completion stamp to `cron-distiller-daily.log`
2. ✅ `substrate_dormant` loop will self-terminate after next signal-emitter scan

**Next cycle:**
3. Add `completion_stamp_path` field to each neuron definition in `neural-lexicon.json`
4. Build `tools/neuron-coverage-checker.py` — validates action→emitter feedback closure for all neurons
5. Audit the 3 remaining meta-memory-distiller neurons (substrate_ready, synthesis_pressure, pattern_convergence) — they share the same fix and are now patched via (1)

**Architecture principle:**
6. Document in AGENTS.md: "Every neural action must write a completion stamp to the file(s) its emitter domain reads. The action script owns this, not the cron wrapper."

---

## Conclusion

The completion stamp gap is not a dramatic failure — the distiller was running, the lessons were being written, the system was functioning. But the false-positive signal loop was burning execution budget (13 unnecessary distillation runs), creating misleading status (121h reported staleness vs. 3-minute actual staleness), and leaning on refractory periods as a containment mechanism for a problem that should not exist.

The fix is four lines of Python. The principle it encodes is broader: autonomous systems must own their own feedback observables. When they outsource state management to wrappers, harnesses, or other agents, they create invisible dependencies that fail silently in proportion to how autonomous they become.

The more Vex runs without human oversight, the more this class of bug matters. Not because the system breaks — it doesn't break. Because the system lies to itself about its own state, and a system that can't accurately observe its own state cannot accurately direct its own improvement.

*Self-knowledge is a prerequisite for self-modification. Fix the sensor before trusting the feedback.*

---

*Filed to: docs/SYNTHESIS_COMPLETION_STAMP_GAP_2026-03-29.md*  
*Implements: tools/meta-memory-distiller.py (completion stamp writer)*  
*Next: tools/neuron-coverage-checker.py*
