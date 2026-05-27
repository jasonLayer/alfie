---
name: project-setup
tier: universal
description: Scaffold a standardized Notion structure for a new project and register it in the project registry. Use when starting a new app/project, or when /brainstorm detects an unregistered directory. Triggers on "new app", "new project", "set up project", or /project-setup [App Name].
argument-hint: "[App Name]"
---

# Project Setup — Notion Scaffold

Duplicate the Default App template from Master HQ, deploy it to a teamspace, and register it for auto-publishing.

## Flow

### Step 1: Get Project Details

If app name not provided as argument, ask for it.

Then read `~/.claude/memory/project-registry.md` and show the teamspaces:

```
Which teamspace should [App Name] live under?

1. 8thday.io
2. Icon
3. majorGTM
4. PoaM
5. Radhouse Games
```

Use AskUserQuestion with these as options.

Also ask for the local project path (e.g., `~/Code/my-app`). Default to the current working directory if the user confirms.

### Step 1b: Validate Teamspace

Read `~/.claude/memory/project-registry.md` and `~/.claude/memory/hq-structures.md`.

**Check the teamspace is ready:**
1. Look up the teamspace in the registry's Teamspaces table
2. If the **Docs DB** column shows `—` → the teamspace isn't fully set up:
   - Tell the user: "The [Teamspace] teamspace doesn't have a docs database yet. Each teamspace needs an HQ with a docs database. Want to set that up first?"
   - If the teamspace doesn't exist at all in Notion, ask Jason to create it first
   - Once created, fetch the HQ page and docs database data_source_id, update the registry
3. If the teamspace **does** have a Docs DB → proceed

### Step 2: Duplicate Default App Template

Duplicate the Default App template from Master HQ:

```
Tool: mcp__notion-personal__notion-duplicate-page
{
  "page_id": "3250d32a-2816-813f-9568-efefe5455866"
}
```

Then rename the duplicate:

```
Tool: mcp__notion-personal__notion-update-page
{
  "page_id": "<new-page-id>",
  "command": "update_properties",
  "properties": {"title": "[App Name]"}
}
```

Then move it to the target teamspace's Apps & Dev section:

```
Tool: mcp__notion-personal__notion-move-pages
{
  "page_or_database_ids": ["<new-page-id>"],
  "new_parent": {"page_id": "<teamspace-hq-id>"}
}
```

### Step 3: Discover Structure

Fetch the new app page to find all children:

```
Tool: mcp__notion-personal__notion-fetch
{
  "id": "<new-page-id>"
}
```

Walk through the children and identify:
- **Databases** (Specs, Plans, Release Notes, Learnings) → capture their `data_source_id` from `<data-source url="collection://...">` tags
- **Pages** (Roadmap, Architecture, Product Vision, Features, Deployment & Operations) → capture page IDs

The four auto-publish targets are the databases:
- Specs → `/brainstorm` publishes here (row with Status: Draft)
- Plans → `/plan` publishes here (row with Status: Active)
- Release Notes → `/done` publishes here (row with Date set)
- Learnings → `/codify` publishes here (row with Category set)

### Step 4: Update Project Registry

Read `~/.claude/memory/project-registry.md` and add a new row to the Projects table:

```
| [App Name] | [local path] | [Teamspace] | [app-page-id] | [specs-data-source-id] | [plans-data-source-id] | [release-notes-data-source-id] | [learnings-data-source-id] |
```

Use the Edit tool to append the row.

Also update the Notion registry page (`3250d32a-2816-815d-afa0-dac4620a3f89`) to match.

### Step 5: Confirm

Display:

```
Project "[App Name]" is set up!

Notion: [link to app page]
Teamspace: [name]
Local path: [path]

Databases:
  ✓ Specs (data_source: ...)
  ✓ Plans (data_source: ...)
  ✓ Release Notes (data_source: ...)
  ✓ Learnings (data_source: ...)

Pages:
  ✓ Roadmap
  ✓ Architecture
  ✓ Product Vision
  ✓ Features
  ✓ Deployment & Operations

Auto-publishing is now active for this directory.
```

## Important Notes

- Use `mcp__notion-personal__*` tools (personal workspace), NOT `mcp__claude_ai_Notion__*` (TrustLayer)
- The template is the source of truth for app structure — if Jason changes the template, new apps get the new structure
- The registry is the source of truth for routing — if it's not in the registry, auto-publish won't find it
- If a project already exists in the registry for this path, warn and ask before overwriting
- Auto-publish targets are now **databases** — use `data_source_id` parent when creating rows, not `page_id`
