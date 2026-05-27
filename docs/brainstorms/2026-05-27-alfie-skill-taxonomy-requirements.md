# Alfie Skill Taxonomy & Config Registry — Requirements
## Tier Names: Universal / Personal / Extension

**Date:** 2026-05-27
**Status:** Draft
**Author:** Jason Reichl

---

## Problem

Alfie has ~60+ skills all sitting in `~/.claude/skills/` with no taxonomy. When a collaborator joins a project and installs Alfie, they get everything — including skills that are hardwired to Jason's personal Notion workspace, Google Calendar, and daily logs. Those skills either fail silently, prompt for config that doesn't exist, or expose internal structure that isn't theirs to use.

Jason works across multiple active projects with collaborators (TrustLayer, MajorGTM, PoaM, 8thday, Jason Reichl). Those collaborators need to pair with Alfie's workflow without needing Jason's personal configuration. And some collaborators will build their own skills that shouldn't pollute Alfie's core.

---

## Core Insight

The taxonomy isn't really about *what a skill does* — it's about *where its config lives*. A skill is "personal" because it requires personal config to function, not because of its subject matter. The taxonomy should follow config location, not intent.

---

## Goals

1. Every skill has a declared `tier` (universal / personal / extension)
2. A config registry maps personal-tier skills to their config requirements and where that config lives
3. A new collaborator can install Alfie, see exactly what skills are universal, and know precisely what to configure to unlock personal-tier skills
4. User extensions (skills a collaborator builds) have a defined home that doesn't pollute core Alfie
5. Skills with optional personal enhancements (e.g., Notion publish) are still classified as team-tier — the enhancement is additive, not required

---

## The Three Tiers

### Tier 1: Universal

**Definition:** Works out of the box on any machine, for any collaborator, with zero personal configuration. May have optional personal enhancements (e.g., auto-publish to Notion if configured) but degrades gracefully when those aren't present.

**Config location:** None required. Optional config at project level (`.claude/`) or team-wide shared config.

**Examples:** `brainstorm`, `plan`, `build`, `qa`, `ship`, `review`, `investigate`, `canary`, `codify`, `done`

### Tier 2: Personal

**Definition:** Requires the user to provide their own credentials, workspace IDs, or personal preferences to function at all. These skills FAIL or are meaningless without personal config — they aren't degraded, they're broken.

**Config location:** `~/.claude/memory/` (personal memory files) or a personal config file at `~/.claude/personal-config.yaml`

**Examples:** `standup` (needs personal Google Calendar + Notion Daily Log DB), `notion-brain` / `notion-read` / `notion-do` (personal Notion workspace), `create-page` / `create-database-row` / `database-query` (personal Notion databases), `weekly` (personal calendar), `landing-report` (personal analytics)

**Behavior for new collaborators:** Personal-tier skills should check for their required config at startup and emit a clear message: `"This skill requires personal configuration. See ~/.claude/personal-config.yaml.template for setup."` rather than failing cryptically.

### Tier 3: Extension

**Definition:** Skills a specific collaborator builds for their own workflow. Not part of core Alfie. Not expected to work for others. Live in a separate directory. Don't auto-load.

**Config location:** The extension skill owns its own config, declared in the skill file.

**Home directory:** `~/.claude/skills-ext/` (separate from `~/.claude/skills/`)

**Discovery:** Alfie is aware extensions exist but doesn't treat them as core. A collaborator's extensions are visible to them only.

---

## Skill Config Registry

A single YAML file at `~/.claude/skill-config-registry.yaml` becomes the source of truth for what personal config each skill needs.

### Schema

```yaml
# ~/.claude/skill-config-registry.yaml

skills:
  standup:
    tier: personal
    config_required:
      - key: notion_daily_log_db
        description: "Notion Daily Log database ID"
        location: "~/.claude/memory/notion-personal-databases.md"
      - key: google_calendar_auth
        description: "Google Calendar MCP authenticated"
        location: "Google Calendar MCP connection"

  notion-brain:
    tier: personal
    config_required:
      - key: notion_personal_workspace
        description: "Personal Notion workspace MCP connected"
        location: "mcp__notion-personal__ MCP connection"

  brainstorm:
    tier: universal
    optional_enhancements:
      - key: notion_specs_db
        description: "If project is registered, auto-publishes to Notion Specs DB"
        location: "~/.claude/memory/project-registry.md"

  plan:
    tier: universal
    optional_enhancements:
      - key: notion_plans_db
        description: "If project is registered, auto-publishes to Notion Plans DB"
        location: "~/.claude/memory/project-registry.md"
```

### Rules
- Every skill gets an entry (even if just `tier: universal` with no other fields)
- `config_required` entries are things the skill NEEDS — missing these = broken
- `optional_enhancements` entries are things that add value but aren't required
- New collaborators get a `skill-config-registry.yaml.template` with personal keys blank

---

## Skill Frontmatter Metadata

Each skill's `SKILL.md` frontmatter gets a `tier:` field:

```yaml
---
name: standup
tier: personal
config: [notion_daily_log_db, google_calendar_auth]
description: ...
---
```

```yaml
---
name: brainstorm
tier: universal
description: ...
---
```

This makes the tier machine-readable and visible to Alfie during skill invocation.

---

## Current Skill Audit — Tier Assignments

### Universal Tier (works for any collaborator, zero personal config)

**Core dev flow:**
`brainstorm`, `plan`, `autoplan`, `build`, `work`, `orchestrate`, `qa`, `qa-only`, `review`, `ship`, `canary`, `codify`, `done`

**Discipline skills:**
`investigate`, `debug` (redirect), `verify`, `receive-review`

**Plan review pipeline:**
`plan-ceo-review`, `plan-eng-review`, `plan-design-review`, `plan-devex-review`, `plan-tune`, `autoplan`

**Design tooling:**
`design-html`, `design-review`, `design-shotgun`, `design-consultation`, `frontend-design`, `implement-design`, `canvas-design`, `theme-factory`

**gstack browser + QA:**
`browse`, `open-gstack-browser`, `setup-browser-cookies`, `pair-agent`

**Utilities:**
`context-save`, `context-restore`, `find`, `search`, `scrape`, `learn`, `health`, `gstack-upgrade`, `gstack`

**Code quality:**
`cso`, `codex`, `benchmark`, `benchmark-models`, `retro`, `office-hours`

**Deployment:**
`land-and-deploy`, `setup-deploy`, `publish`, `ship`

**Documentation:**
`document-generate`, `document-release`, `make-pdf`, `link-pages`, `sync-docs`

**iOS:**
`ios-qa`, `ios-fix`, `ios-clean`, `ios-sync`, `ios-design-review`

**Framer:**
`framer`, `framer-code-components`

**Misc:**
`freeze`, `unfreeze`, `guard`, `careful`, `project-setup`, `skillify`, `devex-review`, `sync-gbrain`, `setup-gbrain`

### Personal Tier (requires per-user config to function)

| Skill | Config Required | Location |
|---|---|---|
| `standup` | Google Calendar auth + Notion Daily Log DB | MCP + `~/.claude/memory/notion-personal-databases.md` |
| `notion-brain` | Personal Notion workspace | `mcp__notion-personal__` MCP |
| `notion-read` | Personal Notion workspace | `mcp__notion-personal__` MCP |
| `notion-do` | Personal Notion workspace | `mcp__notion-personal__` MCP |
| `create-page` | Personal Notion workspace + target DB IDs | `mcp__notion-personal__` MCP |
| `create-database-row` | Personal Notion workspace + target DB IDs | `mcp__notion-personal__` MCP |
| `create-task` | Personal Tasks DB | `~/.claude/memory/notion-personal-databases.md` |
| `database-query` | Personal Notion workspace | `mcp__notion-personal__` MCP |
| `weekly` | Personal calendar + Notion | MCP connections |
| `landing-report` | Personal analytics/data sources | TBD per implementation |
| `invest` | Personal financial/venture data | TBD per implementation |

### Extension Tier (collaborator-built, not core Alfie)

Nothing currently in this tier — this is a new home for skills collaborators create for themselves.

---

## What Goes in gstack vs What's "Official Alfie"

All gstack skills in `~/.claude/skills/` are *available* to any Alfie user. But "available" isn't the same as "part of the Alfie flow narrative."

**Official Alfie skills** (documented in CLAUDE.md + the Dev Flow guide):
The 10-step flow + discipline skills. These are the ones Alfie proactively invokes and routes to.

**Available via gstack** (installed, functional, but not in the primary flow narrative):
Everything else — design tools, iOS tools, codex, retro, office-hours, etc. Collaborators can discover and use these, but Alfie doesn't proactively route to them unless asked.

The distinction matters for onboarding: new collaborators learn the 10-step flow first. The full skill catalog is reference material.

---

## Collaborator Onboarding Pattern

What a new collaborator does to get Alfie working:

1. **Install gstack** — `~/Code/gstack/setup` (installs all team-tier skills)
2. **Get the team-tier skills** — all work immediately, no config needed
3. **Review `~/.claude/skill-config-registry.yaml.template`** — see which personal-tier skills need config
4. **Fill in personal config** — add their own Notion workspace, calendar auth, etc.
5. **Personal-tier skills unlock** as config is provided — they don't all need to be set up at once

---

## Scope Boundaries

**In scope:**
- Taxonomy definition (this doc)
- Config registry YAML schema
- Frontmatter `tier:` field added to all skill SKILL.md files
- Template for collaborator personal config
- Update to CLAUDE.md explaining the taxonomy
- `~/.claude/skills-ext/` directory for extensions

**Out of scope:**
- Building a skill installer CLI
- A public skill marketplace
- Automatic config provisioning / setup wizard
- Per-project `.claude/skills/` directories (project-scoped skills — deferred)

**Deferred:**
- Project-level extensions (skills that live in a repo's `.claude/skills/` and are only available when in that project)
- Config validation tooling (a `/health` check that verifies personal config is set)

---

## Success Criteria

1. A new collaborator can run `/brainstorm`, `/plan`, `/build`, `/qa`, `/review`, `/ship` without any personal configuration
2. When a collaborator runs `/standup` without personal config, they get a clear message explaining what to configure — not a cryptic failure
3. Every skill in `~/.claude/skills/` has a `tier:` in its frontmatter
4. `~/.claude/skill-config-registry.yaml` exists and covers all personal-tier skills
5. A collaborator's custom extension skills live in `~/.claude/skills-ext/` and don't appear in the core Alfie skill list

---

## Open Questions / Assumptions

- **Assumption:** gstack's `./setup` script is the installation mechanism for team-tier skills. Confirm this is the right entry point for collaborators.
- **Open:** Should `weekly` be personal-tier or should it have a team-tier version with no personal data?
- **Open:** `landing-report` and `invest` — need to audit what config they actually require before finalizing their tier.
- **Open:** Project-level extensions (`.claude/skills/` in a repo) are deferred but may become important for MajorGTM customers wanting project-specific skills.
