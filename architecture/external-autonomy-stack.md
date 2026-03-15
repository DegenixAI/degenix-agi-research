# External Autonomy Stack (MVP)

This MVP adds a runnable 3-phase external autonomy stack under `tools/external_autonomy/` with a single CLI entrypoint.

## Architecture

### Phase 1 — Tool-Use Brain + Connector SDK
- `tool_use_brain.py`
  - Routes tasks into: `local_tool`, `connector_call`, `agent_call`
  - Uses composite scoring from stored metrics (`reliability`, `cost`, `latency`)
- `connectors/base.py`
  - Connector base class with retries + exponential backoff + rate-limit pacing
- `connectors/http_json.py`
  - HTTP JSON connector scaffold with transient/429 handling
- Config:
  - Schema: `tools/external_autonomy/config_schema.json`
  - Sample config: `memory/external-autonomy/config.json`

### Phase 2 — Autonomous Build Engine
- `build_engine.py`
  - Ingests capability gaps from `memory/external-autonomy/capability_gaps.json`
  - Generates build contracts
  - Executes one contract into a new artifact file
  - Runs verify command
  - Records contracts/runs to:
    - `memory/external-autonomy/contracts.jsonl`
    - `memory/external-autonomy/runs.jsonl`

### Phase 3 — Live Experiment Runtime + Self-Economy
- `experiments.py`
  - Runs A/B policy trial
  - Picks winner using performance score
  - Writes rollback marker to `memory/external-autonomy/rollback_marker.json`
- `self_economy.py`
  - Computes ROI-style metrics:
    - `time_saved_minutes`
    - `failure_reduction`
    - `tasks_unlocked`
  - Outputs promotion recommendation

## CLI

Entrypoint: `tools/external-autonomy.py`

Subcommands:
- `route`
- `build-run`
- `experiment-run`
- `economy-report`

## Usage examples

```bash
python3 tools/external-autonomy.py route --task "fetch docs" --json
python3 tools/external-autonomy.py build-run --dry-run
python3 tools/external-autonomy.py experiment-run --dry-run
python3 tools/external-autonomy.py economy-report --json
```

Optional real execution (writes artifact + verify):
```bash
python3 tools/external-autonomy.py build-run --json
```
