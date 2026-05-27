---
name: sync-docs
tier: universal
description: After changing a Notion doc, scans its Related Pages cluster for inconsistencies and proposes targeted updates. Prevents stale cross-references when the source of truth changes. Run after any significant edit to a doc that other pages reference.
argument-hint: "[Notion page URL] — the page you just changed (omit to specify interactively)"
---

# Sync Docs — Consistency Propagation

When a document changes, every related document that references the same concepts should stay consistent. This skill finds the drift and proposes targeted fixes — without silently rewriting anything.

---

## Philosophy

**Don't auto-rewrite. Propose and confirm.**

Different docs describe the same system at different levels of abstraction on purpose. The Architecture doc is technical. The Playbook is narrative. Both can describe the dev flow correctly — in different ways. This skill flags *factual* drift (wrong step counts, renamed commands, outdated file paths, removed features) and leaves intentional framing differences alone.

---

## Input

`$ARGUMENTS` — the Notion URL of the page you just changed.

If no URL is provided, ask: "Which page did you just update significantly?"

---

## Step 1: Load the Source Page

Fetch the changed page using `mcp__notion-personal__notion-fetch`.

Extract:
- **Title** and **type** (Guide, Playbook, Reference, etc.)
- **Factual claims**: step counts, command names, file paths, feature lists, skill names, branch names, config values, URLs, process descriptions
- **The Related Pages section** (`## 🔗 Related Pages`) — this is the cluster to check

If there's no Related Pages section, run a Notion search (`mcp__notion-personal__notion-search`) using the page title to find likely related pages. Cap at 5 candidates.

---

## Step 2: Build the Claim Index

From the source page, extract a list of **checkable facts** — concrete, specific statements that other docs might echo or contradict.

Examples of checkable facts:
- "The dev flow has 6 steps: brainstorm → plan → build → review → codify → done"
- "Skills live in `~/.claude/skills/`"
- "The team branch one-liner is `bash <(curl -fsSL ...)`"
- "Auto-publish fires after /brainstorm, /plan, /codify, /done"
- "Worktree isolation: each task gets its own git worktree"
- "TDD Iron Law: no production code without a failing test first"

Don't extract prose opinions or framing — only facts that could be right or wrong in another doc.

---

## Step 3: Scan Related Pages

For each page in the Related Pages cluster, fetch its content and scan for references to the source page's facts.

**For each related page:**

1. **Fetch** the page with `mcp__notion-personal__notion-fetch`
2. **Find overlapping claims** — places where this page mentions the same concepts
3. **Compare** against the source's fact index:
   - ✅ **Consistent** — says the same thing (even in different words) → skip
   - ⚠️ **Drifted** — mentions the concept but with outdated/wrong details → flag
   - 🔇 **Silent** — doesn't mention the concept at all → skip (not a problem)
   - ❌ **Contradicts** — directly states something that conflicts with the source → flag

Only flag Drifted and Contradicts. Silent gaps are fine — not every doc needs to cover everything.

---

## Step 4: Build the Inconsistency Report

Before proposing any changes, present a clear report:

```
📋 Sync Docs — Inconsistency Report

Source: [Page Title]
Pages scanned: N
Issues found: N

DRIFTED:
  ⚠️  [Page Title]
      • Says: "The dev flow has 5 steps"
      • Source says: "6 steps (brainstorm → plan → build → review → codify → done)"
      • Location: "How /build Routes" section

  ⚠️  [Page Title]
      • Says: "Skills live in ~/.claude/commands/"
      • Source says: "Skills live in ~/.claude/skills/"
      • Location: Layer 2 table

CONTRADICTS:
  ❌  [Page Title]
      • Says: "No auto-publish for /done"
      • Source says: "/done auto-publishes to Release Notes DB"
      • Location: Auto-Publish table

CONSISTENT (no changes needed):
  ✅  [Page Title] — all checked facts match
  ✅  [Page Title] — all checked facts match
```

If no issues found:
```
✅ All related pages are consistent with [Page Title]. Nothing to update.
```

---

## Step 5: Propose Fixes

For each flagged issue, propose the exact change:

```
Proposed fix for [Page Title]:

  CURRENT:
  "The dev flow has 5 steps: plan → build → review → codify → done"

  PROPOSED:
  "The dev flow has 6 steps: brainstorm → plan → build → review → codify → done"

  Apply this fix? [y/n/edit]
```

Wait for confirmation on each fix before applying. If the user says "edit", let them dictate the corrected text before writing.

**Never batch-apply all fixes without confirmation.** One at a time, or ask "apply all?" only if all fixes are straightforward corrections with no ambiguity.

---

## Step 6: Apply Confirmed Fixes

For each approved fix, use `mcp__notion-personal__notion-update-page` with:
- `command: "update_content"`
- `content_updates`: the specific old_str → new_str pair

Fetch the page fresh before applying (content may have changed since you read it).

After applying: confirm "✓ Updated [Page Title]"

---

## Step 7: Final Summary

```
✅ Sync Docs — Complete

Source: [Page Title]
Pages scanned: N
Issues found: N
Fixes applied: N
Fixes skipped: N

Updated:
  ✓ [Page Title] — corrected: [brief description of what changed]
  ✓ [Page Title] — corrected: [brief description]

Skipped:
  — [Page Title] — user skipped (review manually)
```

---

## Safety Rules

1. **Never touch TrustLayer** — skip any page whose ancestor path leads to TrustLayer
2. **Propose before writing** — no silent updates, ever
3. **Only fix factual drift** — don't "improve" prose that's just written differently
4. **Fetch fresh before writing** — always re-fetch a page right before applying a fix
5. **Max one update_content call per page** — batch all fixes for a page into one call
6. **If a page has no Related Pages section** — still scan, but note it and suggest running `/link-pages` to build the cluster
7. **If unsure whether something is drift or intentional framing** — flag it as a question, don't auto-classify as an error

---

## Edge Cases

- **Source page has no Related Pages section** → search semantically, limit to 5 candidates, note that `/link-pages` should be run
- **Related page was deleted or 404** → log and skip
- **The "fix" would require rewriting a large section** → flag it as a manual review item, don't propose an in-place edit
- **Two related pages contradict each other (not the source)** → note it in the report but don't resolve it — that's a judgment call for Jason
- **Source page itself seems inconsistent internally** → flag it as a note at the top of the report

---

## When to Run

- After updating a Reference, Playbook, or Guide that other docs cite
- After renaming a skill, command, or file path that appears across multiple docs
- After the dev flow changes (new step added, step renamed, etc.)
- After the Alfie OS architecture changes significantly
- Periodically after a sprint where multiple docs were updated

Not necessary after:
- Adding a new doc that nothing references yet (run `/link-pages` first)
- Updating a Talk Track or Raw manuscript (these aren't cross-referenced as facts)
- Small prose edits that don't change factual content
