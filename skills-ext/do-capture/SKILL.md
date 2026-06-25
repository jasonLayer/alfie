---
name: do-capture
description: Capture items into Jason's personal JR_DO_DB (the dumping ground) and mark them complete. Use whenever Jason says variations of "capture this", "add to my DO", "put it on my list", "remind me to", or any phrasing that means "stash this thought somewhere I'll see it later". Also use when Jason confirms he's done something that's tracked in DO ("I handled X", "X is done"). Two operations only — create_do and mark_done. Never used to triage, prioritize, or process; that happens in Notion by hand.
tier: personal
---

# do-capture

Two-operation skill for the personal DO + Daily Log system. Both operate against `JR_DO_DB`.

**Canonical IDs (always reference these, never re-derive):**
- `JR_DO_DB` (database id): `b05c8c54-8f69-46e1-99b5-b84887e1090d`
- `JR_DO_DB` (page id of the DB block): `82ce296044c74573a22730efead0d6e9`

**Source select values (case-sensitive, must match exactly):**
`Manual`, `Alfie`, `Apple Reminders`, `Sticky`, `Shortcut`, `Apple Notes`, `Rabbit`

---

## Operation 1: `create_do(name, source, context=None)`

Create a new row in JR_DO_DB.

**Inputs:**
- `name` — short verb-phrase string, the title. Required.
- `source` — one of the canonical source values. Required.
- `context` — optional string. If provided, written as the page body so the row carries enough context to act on later. Multi-line strings become multiple paragraph blocks.

**Behavior:**
- `Date Created` is auto-set by Notion (it's a created_time field — do not try to set it).
- `Completed` defaults to unchecked.
- `Date Completed` is left empty.
- No de-dup. If `name` matches an existing row, create a duplicate anyway — Jason wants to see the doubling as a signal to clean up.

**Notion call (via `mcp__notion-personal__notion-create-pages`):**
```
parent: { data_source_id: <JR_DO_DB id> }
properties: {
  "Name": { "title": [{"text": {"content": name}}] },
  "Source": { "select": { "name": source } }
}
children: [ paragraph blocks for context, if provided ]
```

**Return:** the created page's id and url.

**Voice (when invoked by Alfie):**
- Confirm in one short line: "Captured." or "Added to DO."
- Do not list the row fields back — Jason knows what he just captured.
- If `source="Alfie"` (the deferral case), confirm with a touch of warmth: "Parked it. We'll find it tomorrow."

---

## Operation 2: `mark_done(item)`

Mark an existing DO row complete.

**Inputs:**
- `item` — one of:
  - a Notion page id (UUID, hyphens optional),
  - an exact (case-insensitive) `Name` match,
  - a Notion page URL.

**Resolution order:**
1. If it parses as a UUID, treat as page id.
2. Else if it parses as a Notion URL, extract the page id from the URL.
3. Else search JR_DO_DB for `Name` exact match where `Completed = false`. If 0 matches: report and stop. If >1: report the matches and ask Jason which one.

**Behavior:**
- Set `Completed = true`.
- Set `Date Completed = today` (America/Los_Angeles).
- If the row is already completed (`Completed = true`), no-op — do not overwrite `Date Completed`.

**Notion call (via `mcp__notion-personal__notion-update-page`):**
```
page_id: <resolved id>
properties: {
  "Completed": { "checkbox": true },
  "Date Completed": { "date": { "start": "<YYYY-MM-DD>" } }
}
```

**Voice:**
- One short line: "Done — marked complete."
- Don't re-list the item.

---

## Hard rules

- **Source is exact.** No "manual" lowercase, no "iOS Shortcut" — only the seven canonical values.
- **Never edit other fields.** Don't touch `Name` after creation. Don't backfill `Date Created`. Don't write to anything not specified here.
- **Never mutate JR_Plans_DB or JR_Log_DB.** Those live in other skills.
- **Always America/Los_Angeles** for "today" resolution.

---

## When to invoke (Alfie heuristics)

**Invoke `create_do` when:**
- Jason says: "capture this", "add to DO", "put it on my list", "stash this", "remind me to X", "for later", "park this".
- Jason defers something mid-session (LIMB 1): "let's skip that", "not now", "later", "save for later" — propose capture, fire on confirmation.

**Invoke `mark_done` when:**
- Jason says: "I handled X", "X is done", "knocked out X", "shipped X" — and X is plausibly a DO row.
- Don't auto-fire on plan-task completions — those live in JR_Plans_DB, a different system.

**Don't invoke for:**
- Long-form notes — that's not what DO is for.
- Things that are clearly Plans (multi-step features) — use /plan instead.
- Information-only Q&A.
