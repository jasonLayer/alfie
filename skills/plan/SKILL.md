---
name: plan
tier: universal
description: Create a new implementation plan from a spec or idea. Use when the user says "plan this", "create a plan", "let's plan", or provides a spec and wants a structured plan. Runs pre-flight registry check, calls CE plan engine, then auto-publishes to Notion Plans DB.
argument-hint: "[feature or topic to plan]"
---

Create a new plan for: $ARGUMENTS

## Pre-Flight: Project Registry Check

Before starting:
1. Read `~/.claude/memory/project-registry.md` and look up the current working directory
2. If found: note the project name and section IDs for auto-publishing at the end

## Core Engine

**Invoke the Compound Engineering plan skill now:**

```
skill: compound-engineering:ce-plan
args: $ARGUMENTS
```

Follow the CE plan process completely — idea refinement, parallel research agents, SpecFlow analysis, detail level selection, plan writing, and post-generation options. The CE engine handles all of this.

## Post-Plan: Alfie Additions

After the CE plan completes and a plan file has been written:

### Auto-Publish to Notion

If the project registry (from pre-flight) had a match:
1. Confirm briefly: "Publishing to [Project] Plans."
2. Create a database row in the project's Plans DB using `mcp__notion-personal__notion-create-pages` with `data_source_id` parent
3. Title: `[Topic]`, Status: `Active`
4. Include the Notion link in the completion message

If NOT found in the registry: skip silently.

### Flow Recommendation

After Notion publishing, read the completed plan and recommend a specific execution flow. The plan has concrete tasks now — be precise, not generic.

**Assess these signals from the plan:**
- **Task count:** Exact number of implementation tasks
- **Coupling:** Are tasks independent (parallelizable) or tightly coupled (must sequence)?
- **Risk areas:** Auth, payments, data migrations, external APIs, security, or breaking changes?
- **Design surface:** Meaningful UI changes that need browser QA?
- **Post-deploy risk:** Would a regression in production be painful to catch?

**Routing rules:**
- 1-2 tasks OR tightly coupled → use `/work` (sequential, interactive)
- 3+ independent tasks → use `/orchestrate` (parallel worktree subagents)
- High-risk areas OR architectural decisions → run `/autoplan` before building
- Low-risk, well-understood scope → skip `/autoplan`, go straight to `/build`
- Meaningful UI surface → include `/qa`
- Post-deploy risk is real → include `/canary`
- Internal-only change, no user-facing surface → `/review` + `/ship` is enough

**Format the recommendation clearly:**

> **Suggested flow:** [reduced/standard/full]
> `[the specific steps with correct routing — e.g. /work vs /orchestrate]`
> **Why:** [2-3 sentences — task count, coupling, risk areas, what's being skipped and why]

Then remind: "TDD Iron Law applies during `/work`. No production code without a failing test first."
