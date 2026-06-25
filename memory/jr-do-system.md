---
name: jr-do-system
description: Personal DO + Daily Log system for Jason — canonical IDs, schemas, source list, rollup rules, architecture, and operating principles. Always read at session start before invoking do-capture or daily-log.
metadata:
  type: project
---

# Jason's DO + Daily Log System

## What it is

A two-database personal system in Jason's Notion (separate from TrustLayer):

- **`JR_DO_DB`** — the dumping ground. Anything Jason captures lands here. No triage at capture time; processing happens by hand in Notion.
- **`JR_Log_DB`** — the daily mirror. Read-only view of "what was captured / completed / said by Creative Father" on each day.

The system has one rule: **lower friction of capture as close to zero as possible**, then triage in batch later.

## Architecture: Spine + Limbs

- **Spine** = do-capture + daily-log + memory + wire-in. Provides the minimum end-to-end loop.
- **Limbs** add new capture sources (Alfie deferrals, Apple Reminders, sticky-note photos, iOS Shortcut, Apple Notes) and automation (nightly rollup).

Each limb is independent. The spine works without any of them.

## Canonical Notion IDs

| Object                          | Type            | ID                                       |
|---------------------------------|-----------------|------------------------------------------|
| JR HQ (parent page)             | page            | `78d0d32a281682acbb6c8102dd589ee1`       |
| JR_DO_DB                        | database        | `b05c8c54-8f69-46e1-99b5-b84887e1090d`   |
| JR_DO_DB (block id)             | page-of-db      | `82ce296044c74573a22730efead0d6e9`       |
| JR_Log_DB                       | database        | `b3330bfd-876a-46d2-9f36-f94276670db9`   |
| JR_Log_DB (block id)            | page-of-db      | `d3f07e45af804d5c859a7443872a46cd`       |
| JR_Plans_DB                     | database        | `cf15f0de-9315-4d7c-aa8f-49eb0156fb6e`   |
| JR_CreativeFatherQuote_DB       | database        | `25b2c044-2ee6-4f8b-bce8-c5e958e9552c`   |
| JR_CreativeFatherThoughts_DB    | database        | `e2b597c9-f57a-4848-950a-cf0ae3354e20`   |
| JR_DailyLog_DB (ARCHIVED)       | database (old)  | `1ca0d32a-2816-8106-9a57-000be2cde6b0`   |

Notion API uses these as hyphenated UUIDs; the MCP often accepts both forms.

## Schemas

### JR_DO_DB
| Field          | Type          | Notes                                          |
|----------------|---------------|------------------------------------------------|
| Name           | title         | short verb phrase                              |
| Date Created   | created_time  | automatic — never write to this                |
| Completed      | checkbox      | flipped by anything in the completion lane     |
| Date Completed | date          | set when Completed flips to true               |
| Source         | select        | see canonical source list below                |

### JR_Log_DB
| Field | Type   | Notes                                  |
|-------|--------|----------------------------------------|
| Name  | title  | formatted `Mon DD, YYYY`               |
| Date  | date   | the day this page represents (UNIQUE)  |

Page body always contains three section headers in this order: `## Creative Father`, `## Captured Today`, `## Completed Today`.

## Canonical Source values (exact case)

`Manual`, `Alfie`, `Apple Reminders`, `Sticky`, `Shortcut`, `Apple Notes`, `Rabbit`

If a row gets touched by multiple sources, the original wins. Sources are select (single value), not multi-select.

## Rollup rules (daily-log skill)

Trigger: "build today's log" (default today, America/Los_Angeles) or "build the log for {date}".

1. Find or create the JR_Log_DB page where `Date` matches.
2. Replace section bodies — never the page outside the three sections.
3. **Creative Father:** quote (and any thoughts) from the Creative Father DBs for that day.
4. **Captured Today:** JR_DO_DB rows where `Date Created` = target date. Each rendered with checkbox state and `*(Source)*`.
5. **Completed Today:** JR_DO_DB rows where `Date Completed` = target date, PLUS JR_Plans_DB rows completed that day (one line per plan, no task expansion).
6. Read-only mirror — never mutates DO or Plans. If output looks wrong, fix the source row.

## Capture contract (do-capture skill)

- `create_do(name, source, context=None)` — new JR_DO_DB row. `Date Created` is auto. Context goes in page body.
- `mark_done(item)` — flip `Completed=true`, set `Date Completed=today`. No-op if already done.

Both idempotent in spirit. Re-running `create_do` with same name = duplicate (user's signal to clean up).

## Principles

1. **Capture is dumb.** No fields beyond Name+Source+(optional context).
2. **Processing happens in Notion** by hand, in batch.
3. **Completion flows in** from many sources; capture flows in too.
4. **Daily Log is a mirror,** not a controller — never edits its sources.
5. **No prioritization fields.** No tags. No projects. Plans live in JR_Plans_DB.
6. **No notifications.** Pull-based; Jason looks when he wants.

## LIMB 1 — Alfie as source (operational rules)

When Jason defers/skips something in a session, propose capture in one short line. On confirmation, call `create_do(name=summary, source="Alfie", context=<one paragraph of context>)`.

**Triggers:** "later", "not now", "skip", "park that", "save for later", abandoned branch with TODOs, out-of-scope bug spotted mid-session.

**Anti-triggers:** information-only Q&A, mid-flow comparisons, stream-of-consciousness brainstorm, active TDD loop, mid-debug.

**Guardrails:**
- ≤2 capture proposals per turn.
- Don't re-propose the same item in a session.
- One-line proposal: "Capture '{summary}' to DO?" — never block.
- On completion confirmations ("I handled X"), call `mark_done(X)`.

## File locations (Jason's Mac)

- Repo: `~/Code/do-list-system`
- Skills (symlinked): `~/.claude/skills-ext/do-capture`, `~/.claude/skills-ext/daily-log`
- Wire-in: `~/.claude/CLAUDE.md` (one line referencing this memory file)
- This memory: `~/Code/do-list-system/memory/jr-do-system.md`

## Time zone

Always America/Los_Angeles for "today" resolution and for daily filter boundaries.

## Related memories

- [[alfie-flow]] — the broader Alfie dev flow this system plugs into.
- [[document-types]] — Notion document type system. DO and Daily Log are document types 11 and 12 (capture and mirror) outside the normal 12-type spec.
