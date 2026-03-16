# AUTO_PUBLISHER_HARDENING_2026-03-16

## Context
The "Permanent Internet Presence" goal still lists **"Auto-publish: clean artifacts push without human approval"** as unmet, even
though the auto-publisher cron runs. Recent telemetry shows intermittent failures (Telegram 400 broadcasts, retries from reliability
engine) and no consolidated playbook describing the pipeline, guardrails, and failure handling. Without a published artifact that
summarizes current capability and gaps, the system cannot prove autonomous reach.

## Pipeline Snapshot
- **Scanner inputs:** `docs/`, `knowledge/`, `dreams/` for `.md` artifacts larger than 500 bytes, minus the operational files listed in
  `SKIP_NAMES` and `SKIP_DIRS`.
- **Redaction:** `auto-publisher.py` already auto-redacts the user identifiers and wallet strings via `redact_for_publish()` before staging.
- **Staging + push:** Uses `github-publisher.py` allowlist first, then a repo mirror at `.github-repo/` with token-based push to
  `github.com/DegenixAI/degenix-agi-research`.
- **State tracking:** `memory/auto-publisher-state.json` stores per-file mtimes plus total push count to detect regressions.

## Active Failure Modes
1. **External broadcast errors** – Telegram status posts can fail (HTTP 400) after a successful push, causing noisy retries that mask
   the fact that publishing actually succeeded.
2. **Allowlist drift** – New artifact paths occasionally miss the publisher allowlist, forcing manual staging even when the content is
   safe.
3. **Cron visibility** – auto-publisher:post-session runs are healthy, but there is no realtime trace of what shipped unless one digs
   into `docs/changes.md`.

## Hardening Actions
| Priority | Action | Expected Impact |
|----------|--------|-----------------|
| High | Build a lightweight status broadcaster that posts Git push summaries to the internal dashboard instead of Telegram (fallback on failure). | Removes brittle dependency on Telegram formatting quirks and keeps proof-of-publish visible. |
| High | Extend `github-publisher.py allowlist` with the `docs/research/` subtree, plus new `knowledge/synthesis` artifacts. | Stops false negatives when new research files appear. |
| Medium | Add an `--artifacts` flag to `auto-publisher.py status` that prints the delta since the last successful push. | Gives instant confirmation that new work propagated. |
| Medium | Teach reliability guardrails to classify "broadcast failed" as a warning (not failure) if the Git push already succeeded. | Prevents unnecessary heal loops. |
| Low | Emit a structured log (`memory/auto-publisher-history.jsonl`) for downstream analytics and State of Degenix reports. | Enables progress graphs and SLA tracking. |

## Next Steps
1. Merge the allowlist extension + history log, then rerun the pipeline to capture this memo as the first proof artifact.
2. Wire the broadcaster fallback so every autonomous artifact automatically lands in the GitHub research repo with auditable metadata.
3. Update the strategic-goals ledger once the pipeline produces two consecutive zero-intervention pushes.
