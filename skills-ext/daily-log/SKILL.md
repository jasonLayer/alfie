---
name: daily-log
description: Build or refresh Jason's personal JR_Log_DB page for a given day. Use when Jason says "build today's log", "build the log", "build the log for [date]", "refresh today's log", or asks for a daily mirror. Read-only mirror for DO and Plans — never edits JR_DO_DB or JR_Plans_DB. Writes to JR_CreativeFatherThoughts_DB only (creates empty thought + musing rows, cleans up stale empties). Renders four sections: Creative Father (author link + thought row link), Daily Musing (musing row link), Captured Today, Completed Today. Idempotent — running twice produces the same result.
tier: personal
---

# daily-log

Builds (or refreshes) a day page in `JR_Log_DB` — Jason's personal Daily Log.

**Canonical IDs:**
- `JR_Log_DB` (database id): `b3330bfd-876a-46d2-9f36-f94276670db9`
- `JR_Log_DB` (page id of the DB block): `d3f07e45af804d5c859a7443872a46cd`
- `JR_DO_DB`: `b05c8c54-8f69-46e1-99b5-b84887e1090d`
- `JR_Plans_DB`: `cf15f0de-9315-4d7c-aa8f-49eb0156fb6e`
- `JR_CreativeFatherQuote_DB`: `25b2c044-2ee6-4f8b-bce8-c5e958e9552c`
- `JR_CreativeFatherThoughts_DB`: `e2b597c9-f57a-4848-950a-cf0ae3354e20`

**Rotation state file:** `~/.claude/memory/creative-father-rotation.json`

---

## Trigger

- "build today's log" → target date = today (America/Los_Angeles).
- "build the log for {date}" → parse the date.
- "refresh today's log" → same as build today's log; the skill is idempotent.

---

## Process

### 1. Resolve target date
- Default: today in America/Los_Angeles.
- If user provided a date, parse it as LA-time. Accept "May 28", "yesterday", "2026-05-28", "Tuesday".
- Format as `YYYY-MM-DD` (Notion filters) and `Mon DD, YYYY` (page title — no leading zero on day).

---

### 2. Cleanup stale empty Thoughts rows

Before building anything, scan `JR_CreativeFatherThoughts_DB` for rows where:
- `Thoughts` title is empty or blank, AND
- `Created Date` is more than 5 days before target_date

Delete each matching row via the Notion API. This recycles unanswered quotes back into rotation and removes orphaned musing placeholders. Log how many were deleted (silently, no need to surface to Jason unless >0).

---

### 3. Select today's quote (rotation)

Read `~/.claude/memory/creative-father-rotation.json`. Fields used:
- `last_author_url` — the Creative Father relation URL from the last quote shown
- `last_quote_url` — the quote page URL shown most recently

Fetch all rows from `JR_CreativeFatherQuote_DB`. For each row capture:
- `url` — the page URL
- `Quote` — title text (used for display reference only)
- `Creative Father` — relation array (first entry = author page URL)
- `Last Thought Date` — rollup value (null = never responded to)

**Selection algorithm:**
1. Filter out any quote whose `Creative Father` URL matches `last_author_url` (no back-to-back same author). If filtering would leave zero candidates, skip this rule and use all quotes.
2. From remaining, prefer quotes where `Last Thought Date` is null (never responded to). If all have a date, prefer the oldest `Last Thought Date`.
3. Pick the first eligible quote (stable ordering: sort by `Last Thought Date` ascending, nulls first, then by `url` for determinism).

**Exhaustion check:** If every quote in the DB has a non-null `Last Thought Date` AND step 2's "prefer nulls" fallback was used, set `exhausted = true`. After building the page, append the exhaustion message to the Creative Father section (see step 7).

**Save state:** After selecting the quote, write updated `creative-father-rotation.json`:
```json
{
  "last_author_url": "<selected quote's Creative Father URL>",
  "last_quote_url": "<selected quote's page URL>",
  "last_updated": "<target_date>"
}
```

If the rotation file doesn't exist yet, start from scratch (no `last_author_url`).

---

### 4. Find or create the day page

Query `JR_Log_DB` filtered by `Date = target_date`.

- **Found exactly one:** use that page id.
- **Found zero:** create a new page in `JR_Log_DB` with:
  - `Name` = `Mon DD, YYYY`
  - `Date` = target_date
- **Found more than one:** report to Jason and stop. The data is wrong; don't silently pick one.

After finding/creating the page, set its properties:
- `Creative Father Quote` relation → today's selected quote page URL
- `Musing` relation → (updated in step 5 after creating the rows)

---

### 5. Create empty Thoughts rows (idempotent)

Check if `JR_CreativeFatherThoughts_DB` already has rows linked to today's log page (via `Daily Log` relation). If rows already exist for this log, skip creation — this is a refresh run.

If no rows exist yet, create two:

**Row A — Quote Thought:**
```
Thoughts: ""  (empty — Jason fills this in)
Quote: [relation to today's selected quote URL]
Daily Log: [relation to today's log page URL]
```

**Row B — Daily Musing:**
```
Thoughts: ""  (empty — Jason fills this in)
Quote: (empty — no relation)
Daily Log: [relation to today's log page URL]
```

After creating both rows, update the log page's `Musing` relation to include both row URLs.

---

### 6. Gather Captured Today content
- Query `JR_DO_DB` where `Date Created` = target_date (LA day boundary: `on_or_after` start-of-day LA + `on_or_before` end-of-day LA in ISO with UTC offset).
- For each row: `- [ ] {Name} *(Source)*` if `Completed=false`, `- [x] {Name} *(Source)*` if `Completed=true`.
- Sort by `Date Created` ascending.
- If 0 rows: `*Nothing captured today.*`

---

### 7. Gather Completed Today content
- Query `JR_DO_DB` where `Date Completed` = target_date. Render each as `- {Name} *(Source)*`, sorted ascending.
- Query `JR_Plans_DB` for plans with status=Done whose completion timestamp falls within target_date. Render each as `- 📋 {Plan Name}` — one line, no task expansion.
- If both empty: `*Nothing completed today.*`

---

### 8. Write the four sections to the page

The page body has exactly four section headers, always in this order:
1. `## Creative Father`
2. `## Daily Musing`
3. `## Captured Today`
4. `## Completed Today`

For each section: if the header already exists in the page body, replace everything between it and the next header (or end of page) with fresh content. If it doesn't exist, append it. Anything outside the four sections (handwritten notes Jason added) is preserved.

**Creative Father section content:**
```
**{Author Name}** — [open quote ↗]({quote_page_url})
[Add thought ↗]({thought_row_A_url})
```
Where `Author Name` is fetched from the Creative Father profile page (the name field of the relation target). Do NOT copy the quote text into the page body — the quote lives in its own row.

If `exhausted = true`, append after the above:
```
*You've been through all your quotes. Time to add new quotes or discover a new creative father.*
```

If no quote could be selected (DB is empty): render `*No quotes in the Creative Father library yet.*`

**Daily Musing section content:**
```
[Add musing ↗]({thought_row_B_url})
```

**Captured Today / Completed Today:** rendered as described in steps 6–7.

---

### 9. Confirm
One line: `"Today's log: {url}"` or for non-today: `"Log for Jun 20: {url}"`.

---

## What this skill CAN write to

- `JR_Log_DB` — creates/updates log pages and their relation properties
- `JR_CreativeFatherThoughts_DB` — creates empty thought/musing rows; deletes stale empty rows
- `~/.claude/memory/creative-father-rotation.json` — rotation state

## What this skill MUST NOT touch

- `JR_DO_DB` — read only
- `JR_Plans_DB` — read only
- `JR_CreativeFatherQuote_DB` — read only
- `JR_CreativeFathers_DB` — read only

If data looks wrong in DO or Plans, fix the source row — never the Log.

---

## Time zone

All "today" resolution is America/Los_Angeles. Day boundaries: midnight LA = 07:00 UTC (PST) or 08:00 UTC (PDT). Use `on_or_after: <LA midnight in ISO>` and `on_or_before: <LA 23:59:59 in ISO>` for Notion filters.

---

## Idempotency

Running twice on the same day produces the same page state. The four sections are rebuilt from source each run. Thought/musing rows are only created if none are linked to this log yet — re-runs don't create duplicates.

---

## Error modes

- **Notion auth missing:** stop and ask Jason to authenticate notion-personal.
- **Multiple log pages for the same date:** stop, report, ask Jason to clean up.
- **Creative Father DB empty:** render `*No quotes in the Creative Father library yet.*` in the Creative Father section; still create the Daily Musing row and build the other sections.
- **Source DB query fails:** render `*Could not load: {error}*` in that section and continue with the others.
- **Rotation file corrupted/unreadable:** start fresh (treat as if no prior state).
