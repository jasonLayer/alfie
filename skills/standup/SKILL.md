---
name: standup
tier: personal
config: [notion_daily_log_db, google_calendar_auth]
description: Morning briefing — reads yesterday's daily log, today's calendar, and active tasks to produce a quick standup summary. Use at the start of the day or when Jason says "standup", "morning briefing", "what's on today", or "catch me up".
argument-hint: "[optional: date override, e.g. 'yesterday' or '2026-03-15']"
---

# Standup: Morning Briefing

Quick daily briefing that pulls from three sources and gives Jason a clear picture of where things stand.

**Target time:** Under 30 seconds.

## Process

### Step 1: Gather Data (parallel)

Run these three queries simultaneously:

**1a. Yesterday's Daily Log**
Query the Daily Logs DB (`collection://1ca0d32a-2816-8106-9a57-000be2cde6b0`) for the most recent entry.
- Use `mcp__notion-personal__notion-query-database-view` or search by date
- Extract: what was accomplished, any notes, blockers mentioned

**1b. Today's Calendar**
Use `mcp__claude_ai_Google_Calendar__gcal_list_events` to get today's events.
- Extract: meeting times, titles, attendees
- Flag any conflicts or back-to-back blocks

**1c. Active Tasks**
Query the Tasks DB (`collection://1ca0d32a-2816-8170-83e9-000bb490ca5a`) for tasks with active/in-progress status.
- Extract: task name, status, priority
- Sort by priority

### Step 2: Synthesize

Combine the three sources into a briefing. Don't just dump raw data — synthesize.

### Step 3: Output

```
Good morning! Here's your standup:

**Yesterday**
- [accomplishment 1]
- [accomplishment 2]
- [accomplishment 3]

**Today's Schedule**
- 9:00 AM — [meeting/event]
- 11:30 AM — [meeting/event]
- [or "Clear schedule — deep work day"]

**Top 3 Priorities**
1. [highest priority task — why it matters]
2. [second priority]
3. [third priority]

**Blockers**
- [any blockers from yesterday or tasks] or "None — clear path"
```

## After Completion

Record that standup ran today (for autopilot reminders):
```bash
date +%Y-%m-%d > ~/.claude/hooks/.last-standup
```

## Edge Cases

- **No daily log yesterday:** Say "No log from yesterday" and skip that section
- **No calendar events:** Say "Clear schedule — deep work day"
- **No active tasks:** Say "No active tasks — time to plan?"
- **Weekend/holiday:** Adjust to pull from last working day's log
- **Date override:** If Jason provides a date, use that instead of today

## Alfie Touch

Keep it energetic but concise. This is the first thing Jason reads — set the tone for the day. If there's a clear #1 priority, call it out: "Today's about [X] — everything else is secondary."

Reference priorities from CLAUDE.md:
1. Build the ventures
2. Stay healthy
3. Create art
4. Connect

If the schedule is packed with meetings and no build time, gently flag it.
