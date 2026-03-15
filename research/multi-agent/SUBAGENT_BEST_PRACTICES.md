# Subagent Best Practices - AVOIDING POLLING STORMS

## The Problem

When spawning a coding subagent, DON'T do this:

```python
# ❌ WRONG - Creates polling storm
while True:
    status = subagents(action="list")  # Called every few seconds
    sleep(5)
    status = subagents(action="list")  # Called again
    sleep(5)
    # ... hundreds of calls, crashes session
```

This causes:
- Session crashes from tool call overhead
- Wasted tokens on useless status checks
- User has to restart the session

## The Solution

### Option 1: Spawn and Trust (Recommended)

```python
# ✅ CORRECT - Spawn and let OpenClaw handle completion
result = sessions_spawn(
    task="Build XYZ system...",
    label="coding-agent-xyz",
    runTimeoutSeconds=600
)

# DONE. Don't poll. OpenClaw will announce when complete.
# Continue with other work or just wait quietly.
```

**OpenClaw automatically announces subagent completion** - no need to poll!

### Option 2: Use the Subagent Monitor

```bash
# Spawn without waiting
subagent-monitor.py spawn --task "Build system" --label "agent-1"

# Wait intelligently (exponential backoff, not rapid polling)
subagent-monitor.py wait agent-1 --timeout=600

# Or check status occasionally (cached, not hammered)
subagent-monitor.py status agent-1
```

### Option 3: Spawn with --wait Flag

```bash
# Spawns and waits intelligently
subagent-monitor.py spawn --task "Build system" --label "agent-1" --wait
```

## Key Principles

1. **Trust the system**: OpenClaw announces completion, you don't need to poll
2. **Exponential backoff**: If you MUST wait, use increasing intervals (5s → 7s → 10s → 15s...)
3. **Cache results**: Don't call `subagents list` more than once every 5-10 seconds
4. **Set timeouts**: Always use `runTimeoutSeconds` so subagents can't run forever
5. **Kill stuck agents**: Use `subagents kill` if something goes wrong

## Anti-Patterns

❌ **Rapid polling**: Checking status every 5 seconds for 10 minutes = 120 API calls
❌ **Nested waits**: Creating exec processes to sleep, then polling those
❌ **No timeout**: Letting subagents run indefinitely
❌ **Blocking main thread**: Waiting when you could be doing other work

✅ **Right approach**: Spawn, maybe wait intelligently once, trust the completion notification

## When Subagents Complete

You'll see a system message like:
```
[Tue 2026-02-17 04:42 UTC] A background task "coding-agent-xyz" just completed successfully.
```

Then you can fetch the results with:
```python
sessions_history(sessionKey="agent:main:subagent:xxx", limit=50)
```

## Emergency: Kill a Stuck Subagent

```bash
# List active
subagents action=list

# Kill specific one  
subagents action=kill target="agent-label"

# Or via monitor
subagent-monitor.py kill agent-label
```

## The Rule

**One spawn. One wait (optional). One result fetch.**

Not: spawn → poll → poll → poll → poll → crash
