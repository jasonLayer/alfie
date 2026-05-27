---
name: weekly
tier: personal
config: [google_calendar_auth, notion_personal_workspace]
description: Generate weekly HQ updates across all teamspaces. Scans 6 databases per HQ (Plans, Docs, Outputs, Meetings, Updates, Templates), writes structured summary to the right column of each HQ page. Triggers on "weekly update", "update the HQ", "refresh HQ", or runs every Monday.
argument-hint: "[optional: teamspace name or 'all']"
---

# Weekly: HQ Update Generator

Scans all teamspace HQ pages, aggregates activity from the past 7 days across 6 databases per HQ, and writes a structured weekly update to each HQ page's right column.

## HQ Map

| HQ Page | Teamspace | Abbreviation | Page ID |
|---------|-----------|-------------|---------|
| JR HQ | OS | JR | 78d0d32a-2816-82ac-bb6c-8102dd589ee1 |
| mGTM HQ | mGTM | mGTM | 2590d32a-2816-8033-85cb-cf9a2d964df1 |
| PoaM HQ | PoaM | POAM | 2d50d32a-2816-812b-8855-dc198be78feb |
| Radhouse Games HQ | Radhouse Games | RADG | 3080d32a-2816-81ba-88cc-e9a152a56eff |
| StyleIcon HQ | StyleIcon | ICON | 31e0d32a-2816-8114-824a-d7208341ee85 |

## Process

### Step 1: Determine Scope

- If a specific teamspace is named → update only that HQ
- If "all" or no argument → iterate through all 5 HQ pages
- Default date range: last 7 days (overridable)

### Step 2: For Each HQ Page

#### 2a. Load HQ Page
Fetch the HQ page using `mcp__notion-personal__notion-fetch`. Identify:
- The Admin section (contains the 6 databases)
- The right column (where updates go)
- The Mission Statement callout
- The existing "This Week's Update" section (if any)

#### 2b. Scan 6 Databases (Last 7 Days)

Query each database for entries created or modified in the past 7 days:

1. **Plans DB** — active plans, completed plans, new plans
2. **DocsAndResource DB** — new docs, updated docs, by type
3. **AIOUTPUT DB** — new AI outputs, by type (Analyst/Digest/Radar)
4. **Meetings DB** — meetings held, key decisions
5. **Updates & Maintenance** — system changes, maintenance items
6. **Templates DB** — new or modified templates

Use `mcp__notion-personal__notion-query-database-view` for each.

#### 2c. Also Gather Git Activity

For teamspaces with registered projects (check `~/.claude/memory/project-registry.md`):
- Run `git log --oneline --since="7 days ago"` in each project directory
- Summarize: number of commits, key features/fixes

#### 2d. Evaluate Mission Statement

Read the existing Mission Statement callout. Only update if:
- There's concrete evidence the focus has shifted (e.g., a major plan completed, new initiative started)
- Never change it just because — preserve stability

#### 2e. Write the Update

Use this template:

```markdown
# This Week's Update
---
**Week of [Start Date] — [End Date]** — [TeamSpace Name]

## Completed
- [completed plans, shipped features, finished docs]

## In Progress
- [active plans, ongoing work]

## Flags & Attention
- [blockers, risks, items needing Jason's attention]

## By the Numbers
X plans active | Y completed | Z new docs | W meetings

## Looking Ahead
- [what's coming next week based on active plans and tasks]
```

For quiet weeks (< 3 items total), use a simplified single-paragraph format instead.

#### 2f. Apply Update

Use `mcp__notion-personal__notion-update-page` with `update_content` command.
- Only touch the right column "This Week's Update" section
- Never touch: left column, Nav, Workbench, Admin toggle

#### 2g. Log the Update (Optional)

Create an entry in the Updates & Maintenance DB:
- Name: `Weekly Update — [Date]`
- Type: maintenance
- Status: completed

### Step 3: Summary

After all HQ pages are updated:

```
Weekly updates complete!

JR HQ: 3 completed, 5 in progress, 2 flags
mGTM HQ: 1 completed, 3 in progress, 0 flags
PoaM HQ: quiet week (single paragraph)
Radhouse Games HQ: no activity
StyleIcon HQ: 2 completed, 4 in progress, 1 flag
```

## After Completion

Record that weekly ran (for autopilot reminders):
```bash
date +%Y-%m-%d > ~/.claude/hooks/.last-weekly
```

## Guardrails

- **Scan all 6 databases per HQ** — never skip any, even if empty
- **Use relative date queries** — "last 7 days" not hardcoded dates
- **Never touch the left column** — updates go in the right column only
- **Don't overwrite Mission Statement every week** — only when there's real evidence of evolution
- **Don't create duplicate entries** if run twice in the same week — check for existing update first
- **Gracefully handle:** non-conforming pages, empty databases, placeholder mission statements
- **Auto-publish:** After writing to Notion, confirm with link

## Auto-Publish

This skill writes directly to Notion — no separate publish step needed. The output IS the Notion update.
