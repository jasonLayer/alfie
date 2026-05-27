---
name: codify
tier: universal
description: Capture a learning or solution from current work into the knowledge base. Use when the user says "codify this", "capture this learning", "document this solution", or after solving a tricky problem worth preserving. Auto-publishes to Notion Learnings DB.
argument-hint: "[topic or learning to codify]"
---

Codify a learning from the current work: $ARGUMENTS

## Core Engine

**Invoke the Compound Engineering compound skill now:**

```
skill: compound-engineering:ce-compound
args: $ARGUMENTS
```

Follow the CE compound process completely — parallel research subagents (context analyzer, solution extractor, related docs finder, prevention strategist, category classifier), assembly, optional enhancement with specialized agents. The CE engine handles all of this.

## Post-Codify: Alfie Additions

### Auto-Publish to Notion

After the CE compound completes and the solution file has been written:
1. Read `~/.claude/memory/project-registry.md` and look up the current working directory
2. If found:
   - Confirm briefly: "Publishing to [Project] Learnings."
   - Create a database row in the project's Learnings DB using `mcp__notion-personal__notion-create-pages` with `data_source_id` parent
   - Title: `[Topic]`, Category: pick best fit from the solution's category
   - Include the Notion link in the completion message
3. If NOT found in the registry: skip silently.
