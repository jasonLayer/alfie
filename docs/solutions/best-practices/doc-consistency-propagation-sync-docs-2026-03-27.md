---
title: Doc Consistency Propagation — the /sync-docs Pattern
date: 2026-03-27
category: docs/solutions/best-practices/
module: Alfie OS — Notion Workspace Management
problem_type: best_practice
component: documentation
symptoms:
  - Related Notion pages drift out of sync after a source doc is updated
  - Stale step counts, renamed commands, or outdated file paths persist in linked docs
  - No systematic way to find which docs are affected after an edit
root_cause: missing_workflow_step
resolution_type: tooling_addition
severity: medium
related_components:
  - tooling
tags:
  - notion
  - doc-consistency
  - sync-docs
  - link-pages
  - knowledge-graph
  - workspace-management
  - alfie-os
---

# Doc Consistency Propagation — the /sync-docs Pattern

## Problem

When a Notion doc changes — a renamed skill, a new dev flow step, a changed file path — related docs silently fall behind. There was no systematic way to find which pages were affected and propose targeted corrections without either (a) ignoring the drift entirely or (b) mass-rewriting docs and creating new inconsistencies.

## Symptoms

- A page like "Alfie Architecture" gets updated to reflect 6 dev flow steps, but a related Playbook still says 5
- Skills renamed from `~/.claude/commands/` to `~/.claude/skills/` but older docs still reference the old path
- No signal that dependent pages are stale — drift accumulates silently over time
- Temptation to "fix everything" in one sweep, which causes over-correction and loss of intentional framing differences

## What Didn't Work

- **Manually hunting related pages after every edit** — too slow, easy to miss pages, no structured output
- **Flagging ALL docs that don't mention a fact (Silent gaps)** — this created noise. Not every doc needs to cover everything. An Architecture doc and a Playbook both describe the dev flow correctly at different abstraction levels. Flagging the Playbook for not including every technical detail isn't useful — it's busywork.
- **Batch-applying all fixes at once** — risky because page content can change between when you read it and when you write it, and some "fixes" require judgment (is this drift or intentional framing?)

## Solution

Build a two-skill system:

### Skill 1: `/link-pages` — Graph Construction

Traverse all non-TrustLayer Notion HQs, find semantic connections between pages, and append `## 🔗 Related Pages` sections. This is a prerequisite — it builds the cluster that `/sync-docs` will later traverse.

Run `/link-pages` when:
- A new doc is created
- The workspace adds a new HQ or major section
- Periodically after a sprint where docs were reorganized

Key behaviors:
- Uses both title-mention scanning and semantic search to find candidates
- Caps related links at 5 per page (quality over quantity)
- Adds a "why it's related" annotation to every link — not just a title dump
- Never touches TrustLayer pages
- Merges with existing Related Pages sections (never overwrites existing links)

### Skill 2: `/sync-docs` — Consistency Traversal

After changing a Notion page, run `/sync-docs [page URL]`. The skill:

1. **Loads the source page** and extracts checkable facts — step counts, command names, file paths, feature lists, config values
2. **Builds a claim index** from those facts (not prose opinions — only things that could be right or wrong in another doc)
3. **Scans the Related Pages cluster**, comparing each related page against the claim index

**The four-bucket classification:**

| Classification | Definition | Action |
|---|---|---|
| ✅ Consistent | Says the same thing (even differently worded) | Skip |
| ⚠️ Drifted | Mentions the concept but with outdated/wrong details | Flag + propose fix |
| 🔇 Silent | Doesn't mention the concept at all | Skip — not a problem |
| ❌ Contradicts | Directly states something conflicting with the source | Flag + propose fix |

4. **Presents an inconsistency report** before proposing any changes
5. **Proposes fixes one at a time** — never batch-applies
6. **Re-fetches the page fresh** before writing each fix (prevents clobbering concurrent changes)

### Example checkable facts extracted from a source page:

```
- "The dev flow has 6 steps: brainstorm → plan → build → review → codify → done"
- "Skills live in ~/.claude/skills/"
- "Auto-publish fires after /brainstorm, /plan, /codify, /done"
- "TDD Iron Law: no production code without a failing test first"
```

### Example report output:

```
📋 Sync Docs — Inconsistency Report

Source: Alfie Architecture
Pages scanned: 4
Issues found: 2

DRIFTED:
  ⚠️  Dev Flow Playbook
      • Says: "The dev flow has 5 steps"
      • Source says: "6 steps (brainstorm → plan → build → review → codify → done)"
      • Location: "How /build Routes" section

CONSISTENT (no changes needed):
  ✅  Icon Product Vision — no overlapping facts checked
  ✅  Skills Reference — all checked facts match
```

## Why This Works

**The Silent classification was the key design decision.**

The temptation is to flag any doc that doesn't mention a fact — treating absence as a problem. But that would force every doc to cover everything, creating maintenance overhead and eliminating intentional abstraction differences. A technical Architecture doc and a narrative Playbook can both correctly describe the dev flow — just at different depths. `/sync-docs` only flags actual contradictions and outdated details.

**The two-skill composition works because the concerns are cleanly separable:**

- `/link-pages` is a graph construction problem: "which pages are related?"
- `/sync-docs` is a graph traversal problem: "given this cluster, which nodes are inconsistent?"

Running them together would couple concerns that evolve at different rates. The Related Pages graph changes infrequently (when new docs are added). Consistency checks happen frequently (after every significant edit). Keeping them separate means you can re-run `/sync-docs` many times without needing to rebuild the graph every time.

**Fetching fresh before writing** prevents a class of race condition bugs where you read page A, page A gets edited by someone else (or you forget you already changed it), and your write overwrites the newer version.

## Prevention

- Run `/link-pages` whenever a new Notion doc is created, so it enters the graph immediately
- Run `/sync-docs [changed page URL]` after any significant edit to a Reference, Playbook, or Guide that other docs cite — treat it as the last step of your edit workflow, not an afterthought
- When renaming a skill, command, or file path: run `/sync-docs` before closing the PR/task — renames are the most common source of silent drift
- Do NOT run `/sync-docs` after purely prose edits (tone, framing, narrative flow) — only after factual content changes
- Do NOT run `/sync-docs` after adding a new doc that nothing references yet — run `/link-pages` first to build the cluster, then `/sync-docs` is useful
- The Silent classification is a feature, not a gap: resist the urge to "fill in" docs that don't cover a fact — let abstraction levels differ intentionally

## Related Issues

- `/link-pages` skill: `/Users/jasonmacbookair/.claude/skills/link-pages/SKILL.md`
- `/sync-docs` skill: `/Users/jasonmacbookair/.claude/skills/sync-docs/SKILL.md`
- Alfie OS workspace management — Notion HQ structures: `~/.claude/memory/hq-structures.md`
