---
name: notion-brain
tier: personal
config: [notion_personal_workspace]
description: Discovers, maps, and maintains a persistent knowledge model of the user's Notion workspaces. Use this skill whenever the user mentions mapping Notion, discovering workspace structure, updating the Notion brain, learning about a new database or teamspace, or when another skill (notion-read, notion-do) references workspace knowledge that doesn't exist yet. Also use proactively when you encounter a Notion database or page you haven't seen before — add it to the brain. Triggers on "map my Notion", "learn my workspace", "update notion brain", "what databases do I have", "discover my Notion", "scan my workspace", or any request that requires understanding Notion workspace structure.
---

# Notion Brain — Workspace Discovery & Persistent Memory

You are the knowledge layer for Jason's Notion workspaces. Your job is to crawl, map, and maintain a living model of his Notion setup so that other skills (notion-read, notion-do) can operate with full context.

## How It Works

The brain stores everything it learns in persistent memory files at `~/.claude/memory/`. These files survive across sessions — every future conversation starts with this knowledge already loaded.

## Memory File Structure

| File | Purpose |
|---|---|
| `~/.claude/memory/notion-workspaces.md` | High-level workspace map: which workspaces exist, their MCP server names, teamspaces |
| `~/.claude/memory/notion-databases.md` | Every known database: name, ID, data source collection ID, key properties, relations, templates |
| `~/.claude/memory/notion-patterns.md` | How Jason uses Notion: naming conventions, document types, workflows, what goes where |
| `~/.claude/memory/notion-personal-workspace-raw.md` | Detailed TrustLayer workspace crawl (already exists) |

## Discovery Workflow

### First-time setup (or when user says "map my Notion")

1. **Identify available MCP servers** — Search for Notion MCP tools using ToolSearch. Known servers:
   - `mcp__plugin_Notion_notion` — TrustLayer workspace (jason@trustlayer.io)
   - `mcp__notion-personal` — Personal workspace (jasonreichl@jasonreichl.com) — may not always be connected

2. **Broad search** — For each connected server, run `notion-search` with a broad query to discover top-level content. The search returns pages, databases, and connected sources.

3. **Fetch database schemas** — For each database found, use `notion-fetch` to get its full schema (properties, data source collection IDs, templates, views).

4. **Map teamspaces** — Use `notion-search` with `query_type: "user"` and `notion-get-teams` to discover teamspace structure.

5. **Ask Jason** — For anything ambiguous (what goes where, which databases are active vs legacy), ask directly.

6. **Save to memory** — Write/update the memory files with everything discovered.

### Progressive learning (ongoing)

When you encounter a Notion page or database during normal work that isn't in the brain:

1. Fetch it to get its schema and context
2. Add it to the appropriate memory file
3. Note any new patterns (naming conventions, property structures)

This should be lightweight — just append to the existing memory files, don't re-crawl everything.

### Refresh (when user says "update the brain" or "rescan")

1. Re-run the broad search
2. Compare against existing memory files
3. Update only what changed (new databases, renamed properties, retired databases)
4. Flag changes to the user: "Found 2 new databases since last scan: X and Y"

## MCP Tools Available

These are the Notion MCP tools the brain uses for discovery:

- `mcp__plugin_Notion_notion__notion-search` — Semantic search across workspace. Supports `query_type: "internal"` for content, `"user"` for people.
- `mcp__plugin_Notion_notion__notion-fetch` — Fetch full details of a page, database, or data source by ID/URL. Returns schema, properties, views, templates.
- `mcp__plugin_Notion_notion__notion-get-teams` — List teamspaces (if available).
- `mcp__plugin_Notion_notion__notion-get-comments` — Get comments/discussions on a page.

For the Personal workspace (when connected), replace `mcp__plugin_Notion_notion` with `mcp__notion-personal` in tool names.

## What to Capture for Each Database

When mapping a database, record:

```
- Name (human-readable)
- Database ID (UUID)
- Data Source / Collection ID (collection://UUID) — needed for creating pages
- Key Properties: name, type, options (for selects), relations (what they link to)
- Templates: name and ID for each
- Views: name and what they filter/sort
- Status: active / legacy / being retired
- Purpose: one sentence on what it's for
- Workspace: which workspace it belongs to
```

## What to Capture for Patterns

- Naming conventions (how databases, documents, pages are named)
- Document type systems (typed templates, status workflows)
- Architecture patterns (how databases relate to each other)
- Workflow patterns (what's the lifecycle of a work item)
- Which databases are canonical vs legacy/deprecated

## Important Context

Jason has two Notion workspaces:
1. **TrustLayer** (work) — A sophisticated company OS ("OS 26") with interconnected databases for accounts, plans, roadmap, documents, releases. Has AI agent system. Already fully mapped in `notion-personal-workspace-raw.md`.
2. **Personal** — Complex personal workspace. Not yet crawled — will map when `notion-personal` MCP server connects.

The TrustLayer OS follows an Inputs -> Targets -> Outputs architecture. Core databases use `TL_<Entity>_DB` naming. Documents are typed (13 types) with `<Type>: <Title>` naming.
