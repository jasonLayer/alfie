---
name: publish
tier: universal
description: Manually publish a local file to the correct Notion section for the current project. Use as a fallback when auto-publish didn't run, or for standalone docs. Triggers on "/publish", "publish to Notion", "push this to Notion".
argument-hint: "[file-path]"
---

# Publish — Manual Notion Publishing

Publish a local file to the correct Notion section based on the project registry. This is the manual fallback for when auto-publish didn't trigger (e.g., standalone docs, retroactive publishing).

## Flow

### Step 1: Determine What to Publish

If a file path is provided as argument, use it. Otherwise:
1. Check if there's a recently created/modified file in `docs/` (specs, plans, solutions)
2. If multiple candidates, ask the user which one using AskUserQuestion
3. Read the file content

### Step 2: Detect Project

Read `~/.claude/memory/project-registry.md` and look up the current working directory in the Projects table.

If not found:
```
This directory isn't in the project registry. Would you like to:
1. Run /project-setup to register it first
2. Publish to a specific teamspace manually (pick from list)
```

### Step 3: Determine Target Section

Map the file's location to a Notion section:

| File Path Contains | Target Section | Page Title Format |
|---|---|---|
| `docs/specs/` | Specs | `Explanation: [Topic] Design` |
| `docs/plans/` | Plans | `Plan: [Topic]` |
| `docs/solutions/` | Learnings | `Reference: [Type] — [Topic]` |
| `docs/RELEASE_LOG.md` | Release Notes | `Reference: Release [date] — [Summary]` |

If the path doesn't match any pattern, ask the user which section to publish to.

Extract the topic from the file's H1 heading or filename.

### Step 4: Confirm and Publish

Show a brief confirmation:
```
Publishing to [Project] → [Section]
Title: "[Page Title]"
```

Then:
1. Create the page using `mcp__notion-personal__notion-create-pages` with the file content as markdown
2. Move it under the target section page ID from the registry using `mcp__notion-personal__notion-move-pages`

### Step 5: Confirm

```
Published! [Page Title]
Notion: [link]
Section: [Project] → [Section]
```

## Important Notes

- Use `mcp__notion-personal__*` tools (personal workspace)
- Follow the 12-document-type naming convention for page titles
- Read the file content before publishing — don't guess
- If the file is very long (>5000 words), note this but publish anyway — Notion handles it fine
- Each publish creates a NEW page — no versioning or updates to existing pages
