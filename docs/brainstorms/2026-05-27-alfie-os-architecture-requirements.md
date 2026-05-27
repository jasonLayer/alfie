# Alfie OS — Architecture Requirements

**Date:** 2026-05-27
**Status:** Draft
**Repo:** `~/Code/os-structure`

---

## Problem

Jason uses gstack (Garry Tan's open-source AI dev workflow tool) as his AI toolchain foundation. It's excellent for the dev workflow layer, but it doesn't know who Jason is, what he's building, or how to coach him. And as a standalone external project, gstack's methodology evolves on Garry's terms — not Jason's.

The gap: there's no single distributable thing that is *Alfie* — combining the dev workflows, the coaching layer, the multi-surface identity, and Jason's specific venture context — that can eventually be handed to a team member or 8thday client.

---

## Goal

Build **Alfie OS** as an independent product in `~/Code/os-structure` — Jason's AI Chief of Staff operating system, distributable to venture teams and eventually to founders and 8thday clients.

Alfie OS is NOT a fork of gstack. It is its own product that uses gstack as a reference and draws from it selectively, via a standing curation practice.

---

## Users

| Phase | Who | What they get |
|-------|-----|---------------|
| Now | Jason | Full Alfie OS for personal use across all ventures |
| Next | Venture teams (majorGTM, Icon, 8thday) | Alfie OS configured for the team's context |
| Eventually | Founders and operators (broader) | Alfie OS as a publishable product |
| Eventually | 8thday clients | Alfie OS as the AI layer in 8thday coaching |

---

## What Alfie OS Is

Alfie OS is the full stack — not just a dev tool:

1. **Dev workflow layer** — The Alfie-native skills for the full flow (brainstorm → plan → build → qa → review → ship → done). Starts by delegating to gstack's CE engine; grows its own equivalents over time.

2. **Coaching layer** — Adlerian psychology, Chief of Staff watch list, wellbeing monitoring, Protect Version Two. gstack has none of this.

3. **Multi-surface identity** — Alfie works across terminal, Notion, email, calendar. Not just a coding tool.

4. **Venture-aware context** — Knows Jason's ventures (majorGTM, Icon, 8thday, POAM), priorities, relationships, and goals. The memory system carries this forward across sessions.

---

## Architecture Decision: Independent Build + Reference Monitoring

**Chosen approach:** Build Alfie OS independently. Keep gstack installed as a tool/dependency for the CE engine skills it provides. Alfie periodically reviews gstack's update stream and surfaces what's worth incorporating — a curated pull, not a mechanical fork sync.

**Why not fork gstack:**
- A fork ties Alfie OS's file structure to gstack's permanently
- Cherry-pick syncing creates merge debt that grows as the products diverge
- Alfie OS's differentiation is the coaching, identity, and curation judgment — not gstack's code

**The dependency:** gstack's CE engine (ce-brainstorm, ce-plan, and related compound-engineering skills) is used by the current Alfie skill wrappers. This dependency is real and acceptable for now. As Alfie OS matures, these skills get replaced with Alfie-native equivalents and the gstack dependency reduces.

---

## The Curation Practice

When gstack ships updates, Alfie reviews the commit stream and surfaces relevant changes:

> "gstack shipped a new `/investigate` flow in v1.44. Here's what it does. Worth porting into Alfie OS?"

Jason makes the call. What to bring over, what to skip, what to build differently. The curation is not automatic — it's a standing practice, and Jason's judgment is the filter.

This practice is part of Alfie OS's product philosophy: Alfie OS reflects *Jason's* taste about what an AI Chief of Staff should do, informed by but not dictated by gstack.

---

## Scope (Now)

- Create the `os-structure` repo structure for Alfie OS
- Move the Alfie-specific skill files (brainstorm, plan, and others) from `~/.claude/skills/` into `os-structure` so they're version-controlled and distributable
- Create a setup script that installs gstack (as a dependency) and layers Alfie OS on top
- Document the curation practice — how Alfie reviews gstack updates

## Scope (Next — venture teams)

- Per-team configuration (venture context, project registry, Notion workspaces)
- Team member onboarding — install Alfie OS, get Alfie configured for the team
- Alfie OS versioning and upgrade path

## Scope (Later — broader distribution)

- Alfie OS as a publicly installable product
- Alfie-native equivalents of the CE engine skills (reducing gstack dependency)
- 8thday client configuration

---

## Out of Scope

- Forking gstack or maintaining a fork
- Automatic syncing of gstack updates (curation is manual + Alfie-assisted)
- Building everything gstack has — Alfie OS selects what belongs in its product identity
- Replacing gstack for users who just want gstack (different products, different audiences)

---

## Success Criteria

- Alfie OS installs cleanly on a fresh machine with one command
- A new team member at majorGTM or Icon can run `/standup`, `/brainstorm`, `/plan`, and get Alfie — not generic Claude Code
- Jason can point to `os-structure` as the canonical definition of what Alfie is
- When gstack ships something relevant, Alfie surfaces it and Jason makes the call within the week

---

## Dependencies and Assumptions

- gstack remains open-source and installable (true today)
- CE engine skills (compound-engineering:ce-brainstorm, ce-plan, etc.) continue to work as the delegation target for now
- `~/Code/os-structure` is the Alfie OS product repo (currently empty — this brainstorm initiates it)
- The Alfie skill wrappers at `~/.claude/skills/brainstorm/` and `~/.claude/skills/plan/` move into `os-structure` as the source of truth
