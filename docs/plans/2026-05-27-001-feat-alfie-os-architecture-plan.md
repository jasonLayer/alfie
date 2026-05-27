---
title: "feat: Bootstrap Alfie OS repo structure"
type: feat
status: active
origin: docs/brainstorms/2026-05-27-alfie-os-architecture-requirements.md
created: 2026-05-27
---

# feat: Bootstrap Alfie OS repo structure

## Summary

Turn `~/Code/os-structure` into a real, installable Alfie OS product. Move the 31 Alfie-native skills out of `~/.claude/skills/` and into this repo as the canonical source of truth. Add a setup script that installs gstack as a dependency and links Alfie OS skills into place. Document the curation practice for tracking gstack updates.

## Problem Frame

Alfie OS currently lives scattered across `~/.claude/` — skills in `~/.claude/skills/`, identity in `~/.claude/CLAUDE.md`, memory in `~/.claude/memory/`, hooks in `~/.claude/hooks/`. It works for one user, but there's no distributable product. `~/Code/os-structure` is empty. A new team member at majorGTM has no path to getting Alfie.

## Requirements

- Alfie OS installs on a fresh machine with one command (see origin)
- A team member can run `/standup`, `/brainstorm`, `/plan` and get Alfie — not generic Claude Code
- `os-structure` is the canonical definition of what Alfie is
- gstack remains a dependency (not a fork) — updated separately

## Key Technical Decisions

- **CLAUDE.md as a parameterized template:** `CLAUDE.md.tmpl` with `{{NAME}}`, `{{VENTURES}}`, `{{EMAIL}}` placeholders. Setup script substitutes values on install. Personal CLAUDE.md is never checked into the repo.
- **Skills stay as flat directories:** Each skill is `skills/<name>/SKILL.md`. Setup script creates symlinks at `~/.claude/skills/<name>/SKILL.md → os-structure/skills/<name>/SKILL.md` — same pattern gstack uses.
- **gstack installed as a dependency by setup:** Setup script git-clones gstack to `~/.claude/skills/gstack/` if not present. No fork, no vendor.
- **Personal-tier skills included but flagged:** All 31 Alfie-native skills go in. Skills with `tier: personal` in frontmatter require Jason-specific config to function — documented in their SKILL.md.
- **Memory templates not auto-installed:** `memory/` in the repo contains template files. Setup copies them to `~/.claude/memory/` only on first install (no overwrite) — user customizes from there.

---

## Output Structure

```
os-structure/
├── setup                        # Install script (executable)
├── CLAUDE.md.tmpl               # Parameterized CLAUDE.md template
├── skills/                      # All 31 Alfie-native skills
│   ├── brainstorm/SKILL.md
│   ├── plan/SKILL.md
│   ├── build/SKILL.md
│   ├── work/SKILL.md
│   ├── orchestrate/SKILL.md
│   ├── standup/SKILL.md
│   ├── done/SKILL.md
│   ├── codify/SKILL.md
│   ├── verify/SKILL.md
│   ├── receive-review/SKILL.md
│   ├── debug/SKILL.md
│   ├── notion-brain/SKILL.md
│   ├── notion-read/SKILL.md
│   ├── notion-do/SKILL.md
│   ├── create-page/SKILL.md
│   ├── create-database-row/SKILL.md
│   ├── create-task/SKILL.md
│   ├── database-query/SKILL.md
│   ├── find/SKILL.md
│   ├── search/SKILL.md
│   ├── publish/SKILL.md
│   ├── link-pages/SKILL.md
│   ├── sync-docs/SKILL.md
│   ├── project-setup/SKILL.md
│   ├── weekly/SKILL.md
│   ├── canvas-design/SKILL.md
│   ├── frontend-design/SKILL.md
│   ├── implement-design/SKILL.md
│   ├── framer/SKILL.md
│   ├── framer-code-components/SKILL.md
│   ├── theme-factory/SKILL.md
│   └── invest/SKILL.md
├── memory/                      # Identity templates (not auto-overwritten)
│   ├── alfie-identity.md
│   ├── alfie-voice.md
│   ├── alfie-chief-of-staff.md
│   └── alfie-sync-rule.md
├── docs/
│   ├── brainstorms/
│   ├── plans/
│   └── CURATION.md              # How to track and pull gstack updates
└── README.md
```

---

## Implementation Units

### U1. Create repo skeleton and CLAUDE.md template

**Goal:** Establish the directory structure and extract CLAUDE.md into a distributable template.

**Requirements:** Installs cleanly, CLAUDE.md is parameterized not personal.

**Dependencies:** None.

**Files:**
- `README.md` (create)
- `CLAUDE.md.tmpl` (create — extracted from `~/.claude/CLAUDE.md` with personal values replaced by `{{NAME}}`, `{{EMAIL}}`, `{{VENTURES}}` placeholders)
- `skills/` (create empty dir)
- `memory/` (create empty dir)

**Approach:** Extract `~/.claude/CLAUDE.md` as the template base. Replace Jason-specific values (name, email, venture names, Notion IDs) with `{{PLACEHOLDER}}` tokens. The setup script will substitute these on install using sed or envsubst. Keep everything else — the Alfie identity, voice rules, dev flow, Warp tab naming — as-is since those are Alfie OS core, not personal.

**Test scenarios:**
- `CLAUDE.md.tmpl` contains `{{NAME}}` and `{{VENTURES}}` tokens, not literal "Jason Reichl"
- `README.md` exists and explains what Alfie OS is and how to install it
- `skills/` and `memory/` directories exist

**Verification:** `ls os-structure/` shows expected structure; `grep '{{' CLAUDE.md.tmpl` shows placeholder tokens.

---

### U2. Move Alfie-native skills into repo

**Goal:** Copy all 31 Alfie-native skills from `~/.claude/skills/` into `os-structure/skills/` as the new source of truth.

**Requirements:** Skills must work identically after move — symlinks in `~/.claude/skills/` point here.

**Dependencies:** U1 (skills/ dir exists).

**Files:**
- `skills/<name>/SKILL.md` for all 31 skills (copy from `~/.claude/skills/<name>/SKILL.md`)

**Approach:** Copy each skill's SKILL.md into the repo. After copying, verify content is identical. The `~/.claude/skills/<name>/SKILL.md` symlinks will be updated by the setup script (U3) to point to this repo instead of being standalone files.

**Skills to copy:**
brainstorm, plan, build, work, orchestrate, standup, done, codify, verify, receive-review, debug, notion-brain, notion-read, notion-do, create-page, create-database-row, create-task, database-query, find, search, publish, link-pages, sync-docs, project-setup, weekly, canvas-design, frontend-design, implement-design, framer, framer-code-components, theme-factory, invest

**Test scenarios:**
- All 31 `skills/<name>/SKILL.md` files exist in the repo
- Content matches current `~/.claude/skills/<name>/SKILL.md` (no accidental truncation)
- Each skill file has valid SKILL.md frontmatter (`name:` field present)

**Verification:** `ls skills/ | wc -l` returns 31; spot-check 3 skills for content integrity.

---

### U3. Write the setup script

**Goal:** One-command install — clones gstack if needed, links Alfie OS skills into `~/.claude/skills/`, copies memory templates (no overwrite), substitutes CLAUDE.md template.

**Requirements:** Works on a fresh Mac with Claude Code installed. Idempotent — safe to re-run.

**Dependencies:** U1, U2.

**Files:**
- `setup` (create — executable bash script)

**Approach:**

```
setup does:
1. Check Claude Code is installed (~/.claude/ exists)
2. Install gstack if not present:
   - If ~/.claude/skills/gstack/ doesn't exist:
     git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
     cd ~/.claude/skills/gstack && ./setup --quiet
3. For each skill in os-structure/skills/:
   - Create ~/.claude/skills/<name>/ if not exists
   - Symlink ~/.claude/skills/<name>/SKILL.md → <abs-path>/skills/<name>/SKILL.md
4. Copy memory templates to ~/.claude/memory/ (skip if file already exists)
5. If ~/.claude/CLAUDE.md doesn't exist:
   - Prompt for NAME, EMAIL, VENTURES
   - Run envsubst on CLAUDE.md.tmpl → ~/.claude/CLAUDE.md
   Else: skip (don't overwrite existing personal CLAUDE.md)
6. Print success summary
```

**Test scenarios:**
- Running `./setup` on a clean state creates all 31 skill symlinks in `~/.claude/skills/`
- Re-running `./setup` is idempotent — no errors, no duplicate state
- Memory files are copied to `~/.claude/memory/` only if they don't already exist
- If `~/.claude/CLAUDE.md` already exists, setup does NOT overwrite it
- gstack install is skipped if `~/.claude/skills/gstack/` already exists

**Verification:** After `./setup`, running `claude` and typing `/brainstorm` invokes the Alfie brainstorm skill from this repo.

---

### U4. Write the curation practice doc

**Goal:** Document how Alfie reviews gstack updates and surfaces relevant changes — the standing practice that keeps Alfie OS informed without forking.

**Requirements:** Clear enough that the practice can be followed consistently across sessions.

**Dependencies:** None (can be written in parallel with U1-U3).

**Files:**
- `docs/CURATION.md` (create)

**Approach:** Document:
1. How often to check gstack (when starting a new project, or monthly)
2. The command to see what gstack shipped: `git -C ~/.claude/skills/gstack log --oneline origin/main ^HEAD` (shows commits ahead of local)
3. Alfie's role: read the diff, summarize what's new, flag anything that looks worth porting
4. Jason's role: make the call — bring it in, skip it, or build a better version
5. How to incorporate: copy the relevant SKILL.md section into the Alfie OS skill, commit to `jasonLayer/alfie-os`
6. What NOT to auto-pull: gstack's CE engine internals, host configs, browse binary — those are gstack-specific

**Test scenarios:**
- `docs/CURATION.md` exists and is readable
- Includes the specific git command to check gstack's update log
- Clearly separates Alfie's role (surface) from Jason's role (decide)

**Verification:** Someone new to the repo can read CURATION.md and follow the curation process without additional explanation.

---

## Scope Boundaries

### In scope (this plan)
- All 4 implementation units above
- Works for Jason's personal install on one Mac

### Deferred to Follow-Up Work
- Per-team configuration (venture context, Notion workspace IDs per team member)
- Alfie OS versioning and upgrade path (`alfie-upgrade` skill)
- `README.md` install instructions for team members beyond Jason
- Native equivalents of CE engine skills (reducing gstack dependency)

### Out of scope
- Forking gstack
- Modifying gstack in any way
- Windows or Linux support (macOS only for now)

---

## Risks & Dependencies

- **gstack's CE engine:** brainstorm and plan skills delegate to `compound-engineering:ce-*` from the gstack plugin marketplace. If that plugin changes or breaks, those skills degrade. Acceptable for now — noted as a future migration target.
- **Symlink vs copy:** Setup uses symlinks so edits to `os-structure/skills/` are immediately live. Risk: if the repo is moved or deleted, all skills break. Mitigated by keeping the repo at a stable path (`~/Code/os-structure`).
- **CLAUDE.md template drift:** If the personal `~/.claude/CLAUDE.md` is updated (as happened today with the dev flow changes), the template needs to be kept in sync manually. Mitigated by the curation practice — same discipline applied to CLAUDE.md as to gstack skills.
