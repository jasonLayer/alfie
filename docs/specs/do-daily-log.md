# DO + Daily Log — Design Spec

**Owner:** Jason Reichl
**Created:** 2026-05-29
**Status:** Draft → moving to Active once spine ships

---

## Philosophy

**DO is a dumping ground, not a PM tool.**

- **Capture is dumb.** Anything I think of, see, hear, or get nudged on becomes a JR_DO_DB row in seconds — no triage, no fields to fill, no prioritization at the point of entry.
- **Processing happens in Notion.** When I'm in JR HQ, I open the DO database, look at what's piled up, and decide what to do with it: delete, schedule, promote into a Plan, or just check it off.
- **Completion flows in.** Apple Reminders sync, Alfie marking deferred items done, the iOS Shortcut — all of these set `Completed = true` and `Date Completed = today`.
- **The Daily Log is a read-only MIRROR.** It never edits DO, never edits Plans. It just reflects what was captured today, what was completed today, and what Creative Father said today. If the underlying data is wrong, fix it in DO or Plans — not in the Log.

The system has one rule: **lower the friction of capture as close to zero as possible**, then let me triage in batch when I want to.

---

## Architecture (Spine + Limbs)

The spine is the minimum end-to-end loop: capture → store → mirror.

```
                       ┌──────────────┐
                       │  JR_DO_DB    │  ← capture target
                       └──────┬───────┘
                              │ (read-only)
                              ▼
                       ┌──────────────┐
                       │  JR_Log_DB   │  ← daily mirror page
                       └──────────────┘
                              ▲
                              │ (also reads)
              ┌───────────────┼───────────────┐
              │               │               │
      JR_CreativeFather*  JR_Plans_DB    (future limbs)
```

**Spine:** do-capture skill + daily-log skill + memory file + wire-in.

**Limbs** (added independently, in any order, after spine ships):
- LIMB 1 — Alfie source (defer/skip in a session creates a Source=Alfie row)
- LIMB 2 — Apple Reminders two-way sync
- LIMB 3 — Sticky-note photo → DO via vision pipeline
- LIMB 4 — iOS Shortcut (single-tap voice/text capture)
- LIMB 5 — Apple Notes ingestion
- Automation — nightly rollup that ensures today's log page exists by morning

---

## Notion Schemas

### JR_DO_DB
- `Name` — title — short verb phrase, "Email Cohan about Q3 numbers"
- `Date Created` — created_time — automatic; what day the item entered the pile
- `Completed` — checkbox — toggled on by anything in the completion lane
- `Date Completed` — date — set the moment Completed flips to true
- `Source` — select — one of: `Manual`, `Alfie`, `Apple Reminders`, `Sticky`, `Shortcut`, `Apple Notes`, `Rabbit`

The page body is freeform — any context worth keeping (link, quote, sketch, dictated note) goes there. The Daily Log only references the row by title; the body lives in DO.

### JR_Log_DB
- `Name` — title — formatted "May 29, 2026" (no leading zero on day)
- `Date` — date — the day this log represents; used for filtering and uniqueness

The page body holds three sections (always in this order):
1. **Creative Father** — today's quote (and any associated thoughts)
2. **Captured Today** — bulleted list of DO rows where `Date Created` = this day
3. **Completed Today** — bulleted list of DO rows where `Date Completed` = this day, followed by Plans completed that day (plan-level only, one line each)

---

## Source list (canonical)

| Source            | Meaning                                                                 |
|-------------------|-------------------------------------------------------------------------|
| `Manual`          | I typed it directly into the DB                                         |
| `Alfie`           | Created by Alfie when I deferred/skipped something in a session         |
| `Apple Reminders` | Synced from a Reminders list                                            |
| `Sticky`          | Captured from a photo of a physical sticky note                         |
| `Shortcut`        | iOS Shortcut (Siri / lock screen single-tap)                            |
| `Apple Notes`     | Ingested from an Apple Notes capture                                    |
| `Rabbit`          | From the Rabbit r1 capture device                                       |

Sources are a select, not multi-select. If an item is touched by multiple sources, the original source wins.

---

## Daily Log Rollup Rules

Triggered by saying "build today's log" (or "build the log for [date]"). The skill:

1. Resolve the target date (default: today, in America/Los_Angeles).
2. Find or create the JR_Log_DB page where `Date` = target date.
   - If creating, use the page template so structure starts correct.
   - Set Name to `Mon DD, YYYY` (e.g. "May 29, 2026").
3. **Creative Father section** — query JR_CreativeFatherQuote_DB filtered to target date; if a row exists, render the quote + author. Then query JR_CreativeFatherThoughts_DB filtered to target date; if rows exist, render each as a bullet under the quote.
4. **Captured Today section** — query JR_DO_DB where `Date Created` = target date; render each as `- [ ] {Name} *(Source)*` (or `- [x]` if already completed). One bullet per row.
5. **Completed Today section**:
   - JR_DO_DB rows where `Date Completed` = target date: `- {Name} *(Source)*`.
   - JR_Plans_DB rows where status went to Done on target date: `- 📋 {Plan Name}` — single line per plan, no task expansion.
6. The skill writes/replaces these three sections only. Anything else on the page (notes I added by hand) is preserved.

**Read-only invariant:** the daily-log skill must never mutate JR_DO_DB or JR_Plans_DB. If a count looks wrong, fix the source row.

---

## Capture Contract (do-capture skill)

Two operations:

### `create_do(name, source, context=None)`
- Insert a JR_DO_DB row with `Name=name`, `Source=source`.
- `Date Created` is auto-set by Notion (created_time field).
- If `context` is provided, write it as the page body (paragraph block, or multiple if multi-line).
- Return the page id and url so the caller can reference it.

### `mark_done(item)`
- `item` can be a page id, an exact name match, or a DO url.
- Set `Completed = true`, `Date Completed = today (America/Los_Angeles)`.
- If the row is already completed, no-op (don't overwrite `Date Completed`).

Both operations are idempotent in spirit: re-running `create_do` with the same name is allowed (creates a duplicate — that's the user's signal to clean up), but `mark_done` won't bounce the completed date.

---

## Wire-in

- The repo lives at `~/Code/do-list-system`.
- `setup` (shell script at repo root) symlinks `skills-ext/do-capture` and `skills-ext/daily-log` into `~/.claude/skills-ext/`.
- `~/.claude/CLAUDE.md` gets one line pointing at `memory/jr-do-system.md` and noting do-capture + daily-log are always-on for Jason.
- The memory file holds all Notion IDs, schemas, and rules so future Alfie sessions can operate without re-reading the spec.

---

## Open follow-ups (Notion side)

These are one-time setup tasks executed during spine build:

1. Move JR_DO_DB and JR_Log_DB into JR HQ's "Admin (Do Not Edit)" section.
2. Add Nav shortcuts "DO" and "Daily Log" (linked views) in the JR HQ Nav column.
3. Add a JR_Log_DB page template with the three-section skeleton.
4. Archive the old JR_DailyLog_DB as "JR_DailyLog_DB (Archive)". Creative Father quotes are already preserved in their own dedicated DBs.

---

## Non-goals

- **Not a project manager.** No projects, no tags, no dependencies in DO. Plans live in JR_Plans_DB.
- **Not a notes app.** DO holds short rows; long-form thinking goes elsewhere.
- **No prioritization fields.** No priority, importance, urgency. Triage is by eyeball at processing time.
- **No notifications.** The system is pull-based; I look at it when I want to.
- **No retro automation in the Log.** The Log mirrors; it doesn't summarize, score, or judge.
