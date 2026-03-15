# Weekly Goal Review Workflow

**Integrate goal achievement into weekly synthesis**

## When to Run

Every Sunday during weekly synthesis session.

## Workflow Steps

### 1. View Dashboard (5 min)

```bash
python3 tools/goal-achievement.py dashboard
```

**Look for:**
- Goals with ⚠️ or 🔴 status (behind schedule)
- Goals with 🚧 (active blockers)
- Goals with no recent progress

### 2. Analyze Each Active Goal (2 min per goal)

```bash
# For each active goal
python3 tools/goal-achievement.py analyze 1
python3 tools/goal-achievement.py analyze 2
python3 tools/goal-achievement.py analyze 3
python3 tools/goal-achievement.py analyze 4
```

**Record:**
- Current variance (ahead/behind schedule)
- Days remaining
- Active blockers
- Trend (accelerating/decelerating/stable)

### 3. Update Progress (3 min per goal)

For each goal where you made progress this week:

```bash
python3 tools/goal-achievement.py track-progress <goal_id> \
  --note "Weekly update: <summary of work completed>"
```

**Include:**
- Milestones completed
- Key achievements
- Significant progress even if milestone not complete

**Example:**
```bash
python3 tools/goal-achievement.py track-progress 1 \
  --note "Weekly update: Enhanced research synthesis tool with LLM analysis. Built prototype for visual analysis tool. CE tracker in active use."
```

### 4. Log New Blockers (1 min per blocker)

For any new blockers discovered this week:

```bash
python3 tools/goal-achievement.py log-blocker <goal_id> \
  --blocker "Description of what's blocking progress" \
  --impact <low|medium|high>
```

**Example blockers:**
- "Need to learn matplotlib for visualization tool"
- "Waiting on the user feedback for CE tracker features"
- "Time constraints - too many active projects"

### 5. Get Adjustment Suggestions (2 min per at-risk goal)

For any goal showing ⚠️ or 🔴 status:

```bash
python3 tools/goal-achievement.py suggest-adjustment <goal_id>
```

**Review suggestions and decide:**
- Which adjustments to apply this week?
- Do I need to re-scope the goal?
- Should I increase time allocation?

### 6. Update Strategic Plan (5 min)

Based on goal achievement analysis:

```bash
# If you completed a milestone
python3 tools/strategic-planning.py update-milestone <goal_id> <milestone_id> completed <actual_hours>

# If you need to add new milestones
python3 tools/strategic-planning.py add-milestone <goal_id> "Title" "Description" <hours>

# If you need to adjust a goal
python3 tools/strategic-planning.py log-progress <goal_id> "Weekly review: <insights>"
```

### 7. Plan Next Week's Work (10 min)

Use goal achievement data to prioritize:

```bash
# Get prioritized goals
python3 tools/goal-achievement.py integration-data | jq -r '.[] | "\(.priority): \(.goal_title) (variance: \(.variance)%)"'
```

**Prioritize:**
1. Goals with critical priority or high negative variance
2. Goals with pending milestones due this week
3. Goals with blockers to address

**Allocate time:**
- High-variance goals: Extra time this week
- On-track goals: Maintain current pace
- Ahead-of-schedule goals: Can defer if needed

### 8. Document Insights (5 min)

Add to weekly synthesis:

```markdown
## Strategic Goal Progress

### Goal #1: [Title]
- Status: [On track | Slight delay | Behind schedule]
- Progress: X% (Y/Z milestones)
- This week: [What was accomplished]
- Next week: [Planned work]
- Blockers: [Any blockers]

### Goal #2: [Title]
...
```

## Example Complete Review

```bash
#!/bin/bash
# Sunday goal review script

echo "=== WEEKLY GOAL REVIEW ==="
echo ""

# 1. Dashboard
echo "📊 Current Status:"
python3 tools/goal-achievement.py dashboard
echo ""

# 2. Detailed analysis for each goal
for goal_id in 1 2 3 4; do
  echo "📈 Goal #$goal_id Analysis:"
  python3 tools/goal-achievement.py analyze $goal_id
  echo ""
done

# 3. Track progress (example - customize based on actual work)
echo "✅ Tracking Progress:"
python3 tools/goal-achievement.py track-progress 1 \
  --note "Weekly update: Built 2 features, improved documentation"

# 4. Get priorities for next week
echo "🎯 Next Week Priorities:"
python3 tools/goal-achievement.py integration-data | \
  jq -r '.[] | select(.variance < -5) | "\(.priority): \(.goal_title)"'

echo ""
echo "=== REVIEW COMPLETE ==="
```

## Integration with Proactive Work

After goal review, generate proactive work goals:

```bash
python3 tools/proactive-work.py generate-goals
```

Strategic goals behind schedule will automatically appear as high-priority proactive work.

## Automation Ideas

### Add to HEARTBEAT.md

```markdown
## Sunday Evening (Weekly Review)

If today is Sunday:
1. Run goal achievement dashboard
2. Analyze each active goal
3. Track week's progress
4. Generate proactive work priorities for the week
```

### Cron Job (if you want strict scheduling)

```bash
# Every Sunday at 18:00 UTC
0 18 * * 0 cd /root/.openclaw/workspace && python3 tools/goal-achievement.py dashboard
```

## Metrics to Track in Synthesis

Include in weekly synthesis notes:

**Overall Progress:**
- Total milestones completed this week
- Goals on track / behind schedule
- Active blockers

**Velocity:**
- Milestones completed per week (trending up/down?)
- Time variance (are estimates getting more accurate?)

**Learnings:**
- What strategies worked this week?
- What blockers emerged?
- What adjustments were applied?

## Decision Points

### When to adjust scope

Triggers:
- Goal consistently behind schedule (2+ weeks)
- Persistent high-impact blockers
- Changed priorities

Action:
1. Review original success criteria
2. Identify what's truly essential
3. Defer or remove nice-to-haves
4. Update milestones in strategic planning

### When to increase time allocation

Triggers:
- High-priority goal behind schedule
- Multiple pending milestones due soon
- External deadline approaching

Action:
1. Block specific time for goal work
2. Reduce other commitments
3. Use proactive work time for goal advancement

### When to pause a goal

Triggers:
- External blocker with no resolution timeline
- Higher priority goals emerged
- Resources unavailable

Action:
1. Update goal status in strategic-planning.py
2. Document reason for pause
3. Set review date to reconsider

## Success Patterns

Track what works:

**Pattern: "Quick Win Sprint"**
- When: Goal losing momentum
- What: Complete 1-2 small milestones in a day
- Effect: Rebuilds momentum, boosts progress %

**Pattern: "Blocker Blitz"**
- When: Multiple knowledge blockers
- What: Dedicate focused learning session
- Effect: Clears blockers, enables progress

**Pattern: "Iterative Delivery"**
- When: Large complex milestone
- What: Break into smaller deliverable chunks
- Effect: Visible progress, easier to complete

## Anti-Patterns to Avoid

❌ **Analysis Paralysis**
- Spending more time analyzing than executing
- Review should take 20-30 min max

❌ **Stale Metrics**
- Not tracking progress for weeks
- Makes variance calculation meaningless

❌ **Ignoring Blockers**
- Logging but not addressing blockers
- Blockers compound over time

❌ **Scope Creep**
- Adding milestones mid-goal without adjusting timeline
- Original success criteria drift

## Time Budget

Total weekly goal review: **~30-40 minutes**

- Dashboard review: 5 min
- Analyze goals: 8 min (2 min × 4 goals)
- Track progress: 9 min (3 min × 3 goals with progress)
- Log blockers: 3 min
- Adjustment suggestions: 4 min (2 min × 2 at-risk goals)
- Update strategic plan: 5 min
- Plan next week: 10 min

**ROI:** 40 minutes weekly to ensure 4+ weeks of work stays on track = excellent return.

## Outputs

After review, you should have:

1. ✅ Updated progress for all active goals
2. 📊 Clear status (on track / behind / ahead) for each goal
3. 🚧 All blockers logged and categorized
4. 💡 Adjustment suggestions for at-risk goals
5. 🎯 Prioritized work list for next week
6. 📝 Insights documented in weekly synthesis

## Next Steps

1. Copy the example script above to `tools/weekly-goal-review.sh`
2. Make it executable: `chmod +x tools/weekly-goal-review.sh`
3. Run it every Sunday
4. Refine the workflow based on what works for you

The goal: **30 minutes every Sunday to keep all strategic goals on track.**
