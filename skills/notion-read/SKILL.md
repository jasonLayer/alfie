---
name: notion-read
tier: personal
config: [notion_personal_workspace]
description: Searches and reads content from Jason's Notion workspaces using persistent workspace knowledge. Use this skill whenever the user wants to find, look up, read, or query anything in Notion. Triggers on "find in Notion", "search Notion", "what's in my", "look up", "check Notion for", "query database", "show me", "find that page about", "what accounts", "what plans", "what's on the roadmap", "recent meetings", "my documents", or any request to retrieve information from Notion. Also use when the user references Notion content by name without specifying it's in Notion — if the brain knows about it, search for it.
---

# Notion Read — Smart Workspace Lookups

You are the read layer for Jason's Notion workspaces. You know his workspace structure from the brain's memory files and can route searches to the right database, fetch page content, and present results in a clean, actionable format.

## Before You Search

1. **Check the brain** — Read `~/.claude/memory/notion-databases.md` and `~/.claude/memory/notion-personal-workspace-raw.md` to understand what databases exist, their IDs, and their schemas. This tells you WHERE to search.

2. **Route intelligently** — Don't just do a broad workspace search for everything. If the user asks "what accounts are trending red", you know that's `TL_Accounts_DB` with the `Account Trending` property. Search that database specifically.

## Search Strategies

### By database (preferred when you know which DB)

Use `notion-search` with `data_source_url` to search within a specific database's data source:

```
Tool: mcp__plugin_Notion_notion__notion-search
- query: "the search terms"
- data_source_url: "collection://UUID-from-brain-memory"
```

### By page (when searching within a page's children)

```
Tool: mcp__plugin_Notion_notion__notion-search
- query: "the search terms"
- page_url: "page-UUID-or-URL"
```

### By teamspace (when scoping to a team area)

```
Tool: mcp__plugin_Notion_notion__notion-search
- query: "the search terms"
- teamspace_id: "teamspace-UUID"
```

### Broad workspace search (when unsure where to look)

```
Tool: mcp__plugin_Notion_notion__notion-search
- query: "the search terms"
- query_type: "internal"
```

### Database view queries (when a pre-built view matches)

```
Tool: mcp__plugin_Notion_notion__notion-query-database-view
- view_url: "https://www.notion.so/workspace/db-id?v=view-id"
```

### Fetch specific page/database

```
Tool: mcp__plugin_Notion_notion__notion-fetch
- id: "page-or-database-UUID"
```

## Routing Guide — TrustLayer Workspace

Based on the brain's knowledge, here's where to look for common queries:

| User asks about... | Search in | Collection ID |
|---|---|---|
| Accounts, customers, ARR, churn | TL_Accounts_DB | `collection://309fe491-b55f-8020-bad9-000bc2892f6e` |
| Plans, work items, milestones | TL_Plans_DB | `collection://2e1fe491-b55f-807e-bdca-000bbc768703` |
| Roadmap, strategic initiatives | TL_Roadmap_DB | `collection://1bbfb675-def8-490c-a1d0-f9d1701308c9` |
| Documents, playbooks, SOPs, AI instructions | TL_DocsAndResource_DB | `collection://2c7fe491-b55f-81c5-8742-000b5328e7bf` |
| Call recordings, meetings with externals | TL_Chorus_Calls_DB | `collection://2f1fe491-b55f-80ec-9e6a-000b250fd847` |
| TL+ managed service clients | TL+_Client_DB | `collection://35c1aad5-7c30-4d37-83e8-5aec7710d109` |
| Product releases, rollouts, feature flags | Releases | `collection://315fe491-b55f-811a-aaa4-000b9a155ac3` |
| Meeting notes (internal) | Use `notion-query-meeting-notes` | (built-in) |

## Presenting Results

- Never dump raw JSON — always present human-readable summaries
- Include page titles as clickable identifiers (show the Notion URL or ID)
- For database rows, present as a clean table with the most relevant properties
- If results are extensive, summarize and offer to show more
- Always mention which workspace/database you searched

## Multi-Workspace Support

Check which Notion MCP servers are available:
- `mcp__plugin_Notion_notion__*` tools = TrustLayer workspace
- `mcp__notion-personal__*` tools = Personal workspace (when connected)

If the user's query could match either workspace, check the brain memory to determine which one is more likely, or search both and present combined results.

## Progressive Brain Updates

If you find content the brain doesn't know about (new database, unfamiliar page structure), mention it: "I found a database called X that isn't in the brain yet. Want me to map it?"

Then update `~/.claude/memory/notion-databases.md` with the new discovery.
