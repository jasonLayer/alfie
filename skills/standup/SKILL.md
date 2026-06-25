---
name: standup
tier: personal
config: [notion_daily_log_db, google_calendar_auth]
description: Morning briefing — reads yesterday's daily log, today's calendar, and active tasks to produce a quick standup summary. Also catches up any missing daily log pages since the last one was built. Use at the start of the day or when Jason says "standup", "morning briefing", "what's on today", or "catch me up".
argument-hint: "[optional: date override, e.g. 'yesterday' or '2026-03-15']"
---

# Standup: Morning Briefing

Quick daily briefing that pulls from three sources and gives Jason a clear picture of where things stand.

**Target time:** Under 30 seconds.

**Canonical IDs:**
- `JR_Log_DB`: `collection://b3330bfd-876a-46d2-9f36-f94276670db9`
- `JR_Tasks_DB`: `collection://1ca0d32a-2816-8170-83e9-000bb490ca5a`

---

## Step 0: Daily Log Catch-up (run first, before gathering standup data)

Query `JR_Log_DB` for the most recent log entry (sort by `Date` descending, limit 1).

- If the most recent log date is **today** → nothing to do, skip to Step 1.
- If the most recent log date is **before today** → calculate every missing date between that date (exclusive) and today (inclusive). Build them oldest-first by invoking the daily-log skill for each date.

**Invoke daily-log for each missing date:**
Run the full daily-log skill process for that date. Each run is independent. If one fails, log the error and continue with the next date.

**Report at the top of the standup output** (before Yesterday/Schedule/Priorities):
- If logs were built: `*Built logs for: Jun 21, Jun 22, Jun 23, Jun 24*`
- If only today was missing: `*Built today's log.*`
- If nothing was missing: omit this line entirely.
- If >7 dates missing: `*Built 12 missing logs (Jun 10–Jun 24).*` — don't list every date.

If Notion auth is unavailable, skip Step 0 entirely and note "Log catch-up skipped — Notion auth needed" once in the output.

---

## Step 1: Gather Data (parallel)

Run these four queries simultaneously:

**1a. Yesterday's Daily Log**
Query `JR_Log_DB` (`collection://b3330bfd-876a-46d2-9f36-f94276670db9`) for the most recent entry before today.
- Extract: what was captured, what was completed, any thoughts from the Creative Father section.
- If none found: say "No log from yesterday" and skip that section.

**1b. Yesterday's Calendar**
Use `mcp__claude_ai_Google_Calendar__list_events` for yesterday (start = yesterday 00:00 LA, end = yesterday 23:59 LA).
- Extract: all events that actually occurred — meetings, appointments, blocks.
- Filter out all-day blocks, "Busy" blocks, and personal routine events (Wake Up, Get Ready, Eat Breakfast, etc.).
- Keep: any meeting with attendees, named work blocks, calls, creative sessions.

**1c. Today's Calendar**
Use `mcp__claude_ai_Google_Calendar__list_events` for today.
- Extract: meeting times, titles, attendees.
- Flag any conflicts or back-to-back blocks.

**1d. Active Tasks**
Query `JR_Tasks_DB` (`collection://1ca0d32a-2816-8170-83e9-000bb490ca5a`) for tasks with active/in-progress status.
- Extract: task name, status, priority.
- Sort by priority.

---

## Step 2: Synthesize

Combine the four sources into a briefing. Don't dump raw data — synthesize.

**Cross-reference yesterday's calendar vs. yesterday's log:** identify any calendar events with no corresponding DO row or completion. These are the "uncaptured" moments. Flag them in the Yesterday section with a light note — not an accusation, just visibility. Example: "You had a Create Radar session at 12:30pm — nothing captured from it."

---

## Step 3: Output

```
[*Built logs for: Jun 21, Jun 22* ← only shown if catch-up ran]

Good morning! Here's your standup:

**Yesterday**
- [accomplishment 1]
- [accomplishment 2]
- 📅 Create Radar session (12:30pm) — nothing captured  ← uncaptured calendar events

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

After outputting the briefing, ask once:

> *Anything from yesterday worth logging that didn't make it in?*

- If Jason says something → create one or more DO rows: `Source=Alfie`, `Completed=true`, `Date Completed=yesterday`. Confirm with "Logged." Don't re-list the items.
- If Jason says nothing, "nope", "no", or similar → move on silently. Don't push.
- If Jason lists multiple things → create one row per item, confirm once with "Logged [N] items."

---

## After Completion

Record that standup ran today:
```bash
date +%Y-%m-%d > ~/.claude/hooks/.last-standup
```

---

## Edge Cases

- **No daily log yesterday:** Say "No log from yesterday" and skip that section.
- **No calendar events:** Say "Clear schedule — deep work day."
- **No active tasks:** Say "No active tasks — time to plan?"
- **Weekend/holiday:** Adjust to pull from last working day's log.
- **Date override:** If Jason provides a date, use that instead of today.

---

## Alfie Touch

Keep it energetic but concise. This is the first thing Jason reads — set the tone for the day. If there's a clear #1 priority, call it out: "Today's about [X] — everything else is secondary."

Reference priorities from CLAUDE.md:
1. Build the ventures
2. Stay healthy
3. Create art
4. Connect

If the schedule is packed with meetings and no build time, gently flag it.
