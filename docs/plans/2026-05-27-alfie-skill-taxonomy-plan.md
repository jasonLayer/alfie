---
title: Alfie Skill Taxonomy & Config Registry
status: completed
date: 2026-05-27
origin: docs/brainstorms/2026-05-27-alfie-skill-taxonomy-requirements.md
---

# Alfie Skill Taxonomy & Config Registry

## Problem Frame

87 skills sit in `~/.claude/skills/` with no tier distinction. New collaborators get everything — including skills hardwired to personal Notion databases and Google Calendar. Those skills break silently or expose Jason's internal structure. The taxonomy and config registry create a clear contract: Universal skills work for anyone, Personal skills declare what they need, Extension skills have their own home.

**Scope boundary:** This plan covers metadata tagging and registry creation only. No CLI installer, no setup wizard, no per-project `.claude/skills/` directories.

---

## Implementation Units

### Unit 1: Create `~/.claude/skills-ext/` Directory

**File:** `~/.claude/skills-ext/`
**Test file:** None (directory creation, verified by existence check)

Create the extension home directory. Add a `.keep` file so the directory persists in git if it ever gets tracked.

**Acceptance:**
- `~/.claude/skills-ext/` exists
- Contains a `.keep` file with a one-line comment explaining the directory's purpose

---

### Unit 2: Create Skill Config Registry

**File:** `~/.claude/skill-config-registry.yaml`
**Test file:** None (YAML config, human-reviewed)

Create the registry based on the Personal skill audit in the requirements doc. Every skill that requires personal config gets an entry with `tier: personal` and `config_required` keys. Universal skills get a minimal entry with just `tier: universal`. Universal skills with optional Notion enhancements get `optional_enhancements`.

**Personal skills to cover** (from requirements doc audit):

| Skill | Config Required | Location |
|---|---|---|
| `standup` | Google Calendar auth + Notion Daily Log DB | MCP + `~/.claude/memory/notion-personal-databases.md` |
| `notion-brain` | Personal Notion workspace MCP | `mcp__notion-personal__` |
| `notion-read` | Personal Notion workspace MCP | `mcp__notion-personal__` |
| `notion-do` | Personal Notion workspace MCP | `mcp__notion-personal__` |
| `create-page` | Personal Notion workspace + target DB IDs | `mcp__notion-personal__` |
| `create-database-row` | Personal Notion workspace + target DB IDs | `mcp__notion-personal__` |
| `create-task` | Personal Tasks DB | `~/.claude/memory/notion-personal-databases.md` |
| `database-query` | Personal Notion workspace | `mcp__notion-personal__` |
| `weekly` | Personal calendar + Notion | MCP connections |
| `landing-report` | Personal analytics/data sources | TBD |
| `invest` | TrustLayer investment dashboard | TBD |

**Universal skills with optional Notion enhancements:**
`brainstorm`, `plan`, `codify`, `done` — these auto-publish to Notion when project is registered, but degrade gracefully without it.

**Schema to follow** (see origin doc):
```yaml
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
  brainstorm:
    tier: universal
    optional_enhancements:
      - key: notion_specs_db
        description: "If project is registered, auto-publishes to Notion Specs DB"
        location: "~/.claude/memory/project-registry.md"
```

**Acceptance:**
- File exists at `~/.claude/skill-config-registry.yaml`
- All 11 personal skills have entries with `tier: personal` and at least one `config_required` key
- All 4 Notion-publishing universal skills have `optional_enhancements` entries
- All other ~72 skills have minimal `tier: universal` entries
- YAML is valid (parseable)

---

### Unit 3: Add `tier:` Frontmatter to All Skills

**Files:** All `~/.claude/skills/*/SKILL.md` (87 skill files)
**Test file:** None (metadata tagging, verified by grep audit)

Add `tier: universal` or `tier: personal` to the frontmatter of every skill's SKILL.md. For personal skills, also add a `config:` array listing the config keys required.

**Frontmatter shape:**

Universal:
```yaml
---
name: brainstorm
tier: universal
description: ...
---
```

Personal:
```yaml
---
name: standup
tier: personal
config: [notion_daily_log_db, google_calendar_auth]
description: ...
---
```

**Tier assignments** (from requirements doc audit):

*Personal tier (11 skills):*
`standup`, `notion-brain`, `notion-read`, `notion-do`, `create-page`, `create-database-row`, `create-task`, `database-query`, `weekly`, `landing-report`, `invest`

*Universal tier (all others — ~76 skills including):*
Core dev flow: `brainstorm`, `plan`, `autoplan`, `build`, `work`, `orchestrate`, `qa`, `qa-only`, `review`, `ship`, `canary`, `codify`, `done`
Discipline: `investigate`, `debug`, `verify`, `receive-review`
Plan review: `plan-ceo-review`, `plan-design-review`, `plan-eng-review`, `plan-devex-review`, `plan-tune`
Design: `design-html`, `design-review`, `design-shotgun`, `design-consultation`, `frontend-design`, `implement-design`, `canvas-design`, `theme-factory`
Browser/QA: `browse`, `open-gstack-browser`, `setup-browser-cookies`, `connect-chrome`, `pair-agent`
Utilities: `context-save`, `context-restore`, `find`, `search`, `scrape`, `learn`, `health`, `gstack-upgrade`, `gstack`, `_gstack-command`
Code quality: `cso`, `codex`, `benchmark`, `benchmark-models`, `retro`, `office-hours`
Deployment: `land-and-deploy`, `setup-deploy`, `publish`
Documentation: `document-generate`, `document-release`, `make-pdf`, `link-pages`, `sync-docs`
iOS: `ios-qa`, `ios-fix`, `ios-clean`, `ios-sync`, `ios-design-review`
Framer: `framer`, `framer-code-components`
Misc: `freeze`, `unfreeze`, `guard`, `careful`, `project-setup`, `skillify`, `devex-review`, `sync-gbrain`, `setup-gbrain`, `tasks`

**Acceptance:**
- Every `SKILL.md` has a `tier:` field in its frontmatter
- 11 personal-tier skills have a `config:` array
- Universal skills have no `config:` field
- `grep -r "tier:" ~/.claude/skills/` returns 87+ results

---

### Unit 4: Create Personal Config Template

**File:** `~/.claude/personal-config.yaml.template`
**Test file:** None (template file, human-reviewed)

Create the template collaborators copy to set up their personal config. All personal keys are present but blank, with inline comments explaining each.

```yaml
# ~/.claude/personal-config.yaml
# Personal configuration for Alfie personal-tier skills.
# Copy this file to ~/.claude/personal-config.yaml and fill in your values.

notion:
  personal_workspace_mcp: ""  # mcp__notion-personal__ connection name
  daily_log_db: ""            # Notion Daily Log database collection ID
  tasks_db: ""                # Notion Tasks database collection ID

google:
  calendar_mcp: ""            # Google Calendar MCP connection name (mcp__claude_ai_Google_Calendar__)
```

**Acceptance:**
- File exists at `~/.claude/personal-config.yaml.template`
- All config keys from personal skills are represented
- Inline comments explain each key

---

### Unit 5: Update CLAUDE.md with Taxonomy Documentation

**File:** `~/.claude/CLAUDE.md`
**Test file:** None (documentation, human-reviewed)

Add a "Skill Taxonomy" section to CLAUDE.md explaining the three tiers and where to find the registry. Place it in the appropriate context (after the Notion section or before Code Style).

**Content to add:**
```markdown
## Skill Taxonomy

All skills in `~/.claude/skills/` have a `tier:` field in their frontmatter:

- **Universal** — works for any collaborator with zero personal config. Degrades gracefully if optional Notion enhancements aren't configured.
- **Personal** — requires the user's own credentials or workspace IDs. Fails (not degrades) without config. Config requirements are declared in `~/.claude/skill-config-registry.yaml`.
- **Extension** — collaborator-built skills that live in `~/.claude/skills-ext/` and don't auto-load.

New collaborators: run universal skills immediately. For personal skills, see `~/.claude/personal-config.yaml.template`.
```

**Acceptance:**
- CLAUDE.md contains a Skill Taxonomy section
- All three tier names are defined
- Registry file and template file paths are referenced correctly

---

## Sequencing

```
Unit 1 (skills-ext dir)        ← no dependencies
Unit 2 (config registry)       ← requires tier assignments from requirements doc
Unit 3 (tier frontmatter)      ← requires tier assignments from requirements doc
Unit 4 (personal config template) ← requires config keys from Unit 2
Unit 5 (CLAUDE.md update)      ← last, references all artifacts
```

Units 1, 2, and 3 can run in parallel. Unit 4 after Unit 2. Unit 5 last.

---

## Key Decisions

**Decision: Tier follows config location, not subject matter** (see origin)
A skill is Personal because it BREAKS without user config, not because it touches personal topics. `invest` is Personal because it reads TrustLayer-specific data sources. `standup` is Personal because it needs a specific Notion DB ID and Calendar MCP. This framing prevents misclassification of skills that happen to have personal-sounding names.

**Decision: `config:` array in frontmatter mirrors registry keys**
The frontmatter `config:` array is not duplicating the registry — it's a fast lookup for Alfie during invocation (check if skill is callable before loading full registry). The registry is the detailed source of truth.

**Decision: `weekly` classified as Personal, not Universal**
The requirements doc leaves this open. Since `weekly` reads personal calendar + Notion data and would fail without those connections, it belongs in Personal tier. A team-tier version (if ever needed) would be a separate skill.

**Decision: `tasks/` is a nested skill directory structure**
`tasks/` contains sub-skills (`build`, `explain-diff`, `plan`, `setup`). Each sub-skill gets its own `tier:` frontmatter. The parent `tasks/` dir has no SKILL.md and doesn't get tagged directly.

---

## Test Scenarios

**Unit 1:**
- After creation: `ls ~/.claude/skills-ext/` shows `.keep`
- `.keep` file contains a comment about extension skills

**Unit 2:**
- YAML parses without error: `python3 -c "import yaml; yaml.safe_load(open('~/.claude/skill-config-registry.yaml'))"`
- All 11 personal skills are present
- Each personal skill has at least one `config_required` entry with `key`, `description`, `location`

**Unit 3:**
- `grep -l "tier:" ~/.claude/skills/*/SKILL.md | wc -l` equals 87+ (or total SKILL.md count)
- `grep -rh "tier: personal" ~/.claude/skills/*/SKILL.md | wc -l` equals 11
- `grep -rh "config:" ~/.claude/skills/standup/SKILL.md` shows the expected keys
- No universal skill has a `config:` field

**Unit 5:**
- CLAUDE.md contains "Skill Taxonomy"
- All three tier names appear in the section

---

## Risks & Notes

- **87 skill file edits is mechanical but high-volume** — do them in batches by tier (personal 11 first to validate the pattern, then universal en masse)
- **`tasks/` nested structure** — skip `tasks/SKILL.md` (doesn't exist); tag each sub-skill individually
- **`_gstack-command/SKILL.md`** — internal gstack utility, `tier: universal`; its `name: gstack` frontmatter is intentional (it powers the `/gstack` command)
- **`connect-chrome/SKILL.md`** — `name: open-gstack-browser`; `tier: universal`
- **`landing-report` and `invest` config** — listed as "TBD per implementation" in the requirements doc. Add placeholder entries in the registry with `location: "TBD"` rather than omitting them

---

## Deferred

- Config validation tooling (`/health` check that verifies personal config keys are populated)
- Project-level skill extensions (`.claude/skills/` in a repo)
- Skill installer CLI
- Per-skill startup check that emits the "configure me" message (behavioral, not metadata)

## Known Limitation: Gstack-Sourced Frontmatter

55 skills that are symlinks into `~/Code/gstack/` have SKILL.md files auto-generated from
`.tmpl` templates by gstack's build process (`bun run gen:skill-docs`). The `tier:` tags
added during this plan's execution are overwritten on every gstack upgrade.

**Impact:** The 10 personal skills (all regular files, not gstack symlinks) retain `tier: personal`
durably. The 55 universal gstack skills lose `tier: universal` on each upgrade. Since universal
skills require no config, this is a display/discoverability issue only — not a behavioral failure.

**Source of truth:** `~/.claude/skill-config-registry.yaml` is durable and correct.

**Future fix:** Add `tier: universal` to gstack's SKILL.md.tmpl templates and open a PR to
`~/Code/gstack/`. After merging, regenerated files will carry the tag permanently.
