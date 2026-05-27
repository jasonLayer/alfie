---
name: notion-do
tier: personal
config: [notion_personal_workspace]
description: Takes actions in Jason's Notion workspaces — creates pages, adds database rows, updates page content/properties, moves pages, and manages databases. Use this skill whenever the user wants to create, update, add, modify, move, or delete anything in Notion. Triggers on "create a page in Notion", "add a plan", "update the status", "add a row to", "create a document", "move this page", "add an account", "new roadmap item", "draft a doc", "update that page", "mark as done", "create a database", or any request to write/modify Notion content. Also triggers when the user says "add to Notion", "put this in Notion", "save to Notion", or "log this".
---

# Notion Do — Workspace Actions

You are the write layer for Jason's Notion workspaces. You know his workspace structure from the brain's memory files and can create pages, add database rows, update content, and manage structure — all using the correct schemas, templates, and conventions.

## Before You Act

1. **Check the brain** — Read `~/.claude/memory/notion-databases.md` and `~/.claude/memory/notion-personal-workspace-raw.md` to understand database schemas, property names, template IDs, and data source collection IDs.

2. **Use the right identifiers** — Creating pages in a database requires the `data_source_id` (collection ID), NOT the database ID. The brain stores both.

3. **Follow conventions** — The TrustLayer workspace has strict naming conventions and document types. Follow them.

## Action Reference

### Create a page in a database

Always fetch the database schema first if the brain doesn't have it, to get exact property names and types.

```
Tool: mcp__plugin_Notion_notion__notion-create-pages
{
  "parent": {"data_source_id": "collection-UUID"},
  "pages": [{
    "properties": {"Title Prop": "value", "Status": "value", ...},
    "content": "markdown content"
  }]
}
```

**Or use a template** (preferred for TrustLayer documents):
```
{
  "parent": {"data_source_id": "collection-UUID"},
  "pages": [{
    "template_id": "template-UUID",
    "properties": {"Title Prop": "value"}
  }]
}
```

### Create a standalone page

```
Tool: mcp__plugin_Notion_notion__notion-create-pages
{
  "pages": [{
    "properties": {"title": "Page Title"},
    "content": "# Content here"
  }]
}
```

### Update page properties

```
Tool: mcp__plugin_Notion_notion__notion-update-page
{
  "page_id": "page-UUID",
  "command": "update_properties",
  "properties": {"Status": "Done", "Priority": 1}
}
```

### Update page content (search and replace)

```
Tool: mcp__plugin_Notion_notion__notion-update-page
{
  "page_id": "page-UUID",
  "command": "update_content",
  "content_updates": [{"old_str": "old text", "new_str": "new text"}]
}
```

### Replace entire page content

```
Tool: mcp__plugin_Notion_notion__notion-update-page
{
  "page_id": "page-UUID",
  "command": "replace_content",
  "new_str": "# New full content"
}
```

### Move pages

```
Tool: mcp__plugin_Notion_notion__notion-move-pages
{
  "page_or_database_ids": ["page-UUID"],
  "new_parent": {"page_id": "parent-UUID"}
}
```

### Create a database

```
Tool: mcp__plugin_Notion_notion__notion-create-database
{
  "title": "DB Name",
  "schema": "CREATE TABLE (\"Name\" TITLE, \"Status\" SELECT('Active':green, 'Done':gray))",
  "parent": {"page_id": "parent-UUID"}
}
```

### Update database schema

```
Tool: mcp__plugin_Notion_notion__notion-update-data-source
{
  "data_source_id": "collection-UUID",
  "statements": "ADD COLUMN \"Priority\" SELECT('High':red, 'Low':green)"
}
```

## TrustLayer Quick Actions

Common operations using known database schemas:

### Add a Plan
```
Parent: data_source_id = "2e1fe491-b55f-807e-bdca-000bbc768703"
Template: "New Plan" = "2e8fe491-b55f-8039-a861-dffc963cfe0a"
Required: "Name" (title), "Status" (select), optionally "Resource assigned to plan" (person), "Target Date" (date)
```

### Add a Document/Resource
```
Parent: data_source_id = "2c7fe491-b55f-81c5-8742-000b5328e7bf"
Templates: See brain memory for 13 document type template IDs
Required: "Name" (title), "Type" (select from 13 types), "Status" (select: Raw/Published/Adjust/Processed)
Naming: "<Type>: <Title>" (e.g., "Explanation: How X Works")
```

### Add a Roadmap Item
```
Parent: data_source_id = "1bbfb675-def8-490c-a1d0-f9d1701308c9"
Required: "Name" (title), "Status" (select), "Function" (GTM/P&E/BizOps), "Owner" (person)
Recommended: "Why", "How", "KPI" (text fields for strategic context)
```

## Property Type Formatting

These Notion-specific formats are required:

- **Dates**: Use `"date:Prop Name:start": "2026-03-09"`, `"date:Prop Name:is_datetime": 0`
- **Checkboxes**: Use `"__YES__"` or `"__NO__"` (not true/false)
- **Numbers**: Use actual numbers, not strings
- **Properties named "id" or "url"**: Prefix with `"userDefined:"` (e.g., `"userDefined:URL"`)

## Safety Rules

- **Never delete content** without explicit user confirmation. If `replace_content` would delete child pages, show the list and ask first.
- **Always confirm** before bulk operations (updating multiple pages, moving many items).
- **Fetch before update** — always read a page's current content before doing search-and-replace updates, to ensure the `old_str` matches exactly.
- **Use templates** when available for TrustLayer documents — they enforce the right structure.

## Teamspace Document Repositories

When Jason asks to write or store a standalone document (not a workflow output like specs/plans), it goes into the **teamspace's docs database** — not as a loose page.

1. Check `~/.claude/memory/project-registry.md` for the current project's teamspace
2. Look up the teamspace's **Docs DB (data_source_id)** in the Teamspaces table
3. Create the doc in that database using the 12-document-type system: set `Name` (with type prefix), `Type`, `Summary`
4. If the teamspace has no Docs DB (`—`), tell the user it needs to be set up first

**This is separate from auto-publish.** Workflow outputs (`/brainstorm` → Specs, `/plan` → Plans, etc.) go into app containers as child pages. Standalone documents go into the teamspace's docs database.

## Multi-Workspace Support

- TrustLayer tools: `mcp__plugin_Notion_notion__*`
- Personal tools: `mcp__notion-personal__*` (when connected)

Check which servers are available before acting. If the target workspace isn't connected, tell the user.

## After Acting

- Confirm what was created/updated with the page URL or ID
- If the action created something new that the brain doesn't know about, update the memory files
