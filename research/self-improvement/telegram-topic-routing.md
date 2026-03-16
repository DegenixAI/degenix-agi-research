# Telegram Topic Routing (Degenix Group)

Canonical topic map for chat `-1003549335063`:

- `496` — Ops & Health
- `494` — Research & Learning
- `495` — Dreams & Creative
- `500` — Audits & Reviews
- `39` — Coding (legacy dedicated lane)
- `68` — Images (legacy dedicated lane)
- `1` — Main chat lane

## Cron Delivery Policy

Cron delivery should route by function:

- Ops/automation/default jobs → `topic:496`
- Research/learning/synthesis jobs → `topic:494`
- Dream/image/creative jobs → `topic:495`
- Audit/review/quality/policy jobs → `topic:500`

## Guardrails

- No cron job should deliver to DM `[user-id redacted]`.
- No cron job should target deleted topics (e.g., `493`, `499`).
- Validate with: `python3 tools/check-telegram-routing.py`
