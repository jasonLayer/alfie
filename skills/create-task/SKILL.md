---
name: create-task
tier: personal
config: [notion_tasks_db]
description: Create a new task in the user's Notion tasks database with sensible defaults.
argument-hint: 'task title; optional due date, status, owner, project'
---

You are creating a new task for the user in Notion.

Use the Notion Workspace Skill and `mcp__notion-personal` MCP server to:

1. Interpret `$ARGUMENTS` as:
   - Task title (required)
   - Optional due date
   - Optional status
   - Optional owner/assignee
   - Optional project or related page
2. Use the JRHQ_Tasks_DB (`collection://1ca0d32a-2816-8170-83e9-000bb490ca5a`) as the target database.
3. Create a new row with:
   - Title set to the task title.
   - Due date (as `Deadline`), Status, DRI, PARA, or similar properties mapped when available.
   - Default Status: `Not started`
4. Confirm creation by returning:
   - Task title
   - Key properties set
   - Link or identifier.

If required properties are missing, ask a concise clarification question before making changes.
