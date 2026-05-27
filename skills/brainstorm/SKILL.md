---
name: brainstorm
tier: universal
description: Explore what to build through collaborative dialogue before planning. Use before /plan for any new feature, component, or significant change. Triggers on "let's brainstorm", "I want to build", "what should we do about", ambiguous feature requests, or when requirements have multiple valid interpretations.
argument-hint: "[feature idea or problem to explore]"
---

# Brainstorm (Alfie + Compound Engineering)

This skill delegates to the full Compound Engineering brainstorm engine, then adds Alfie-specific steps.

## Pre-Flight: Project Registry Check

Before starting the brainstorm:
1. Read `~/.claude/memory/project-registry.md` and look up the current working directory
2. If NOT found: "This directory isn't registered as a project yet. Want me to run `/project-setup` first?" If the user says yes, invoke `/project-setup` before continuing.
3. If found: note the project name and section IDs for auto-publishing at the end.

## Core Engine

**Invoke the Compound Engineering brainstorm skill now:**

```
skill: compound-engineering:ce-brainstorm
args: $ARGUMENTS
```

Follow the CE brainstorm process completely — all phases, all interaction rules, all output formats. The CE engine handles scope assessment, product pressure testing, collaborative dialogue, approach exploration, requirements capture, and handoff.

## Post-Brainstorm: Alfie Additions

After the CE brainstorm completes and a requirements document has been written:

### Auto-Publish to Notion

If the project registry (from pre-flight) had a match:
1. Confirm briefly: "Publishing to [Project] Specs."
2. Create a database row in the project's Specs DB using `mcp__notion-personal__notion-create-pages` with `data_source_id` parent
3. Title: `[Topic] Design`, Status: `Draft`
4. Include the Notion link in the completion message

If NOT found in the registry: skip silently.

### Flow Recommendation

After Notion publishing, assess what came out of the brainstorm and recommend a tailored execution flow. Don't just say "run the full flow" — give a specific, reasoned recommendation.

**Assess these signals from the brainstorm:**
- **Task count:** How many distinct implementation tasks does this imply?
- **Risk areas:** Does it touch auth, payments, data migrations, external APIs, or security?
- **Design surface:** Is there meaningful UI/UX to validate, or is it mostly backend/logic?
- **Novelty:** Net-new feature, enhancement to existing, or a bug fix?

**Then recommend one of these flows (with your reasoning):**

**Reduced flow** — use when: simple spec, 1-2 tasks, no risky areas, no new UI surface
```
/plan → /work → /review → /ship
```

**Standard flow** — use when: medium complexity, 3-5 tasks, some design surface or moderate risk
```
/plan → /autoplan → /build → /qa → /review → /ship
```

**Full flow** — use when: complex spec, 5+ tasks, high-risk areas, meaningful UI, or post-deploy monitoring warranted
```
/plan → /autoplan → /build → /qa → /review → /ship → /canary
```

Format the recommendation clearly:

> **Suggested flow:** [reduced/standard/full]
> `[the specific steps]`
> **Why:** [1-2 sentences on what drove this recommendation — task count, risk areas, design needs]

Then remind: next step is `/plan <name>` to turn the spec into an implementation plan.
