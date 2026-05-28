# Alfie OS

Alfie is an AI Chief of Staff built on top of [Claude Code](https://claude.ai/code). This repo is the canonical definition of what Alfie is — 31 skills, identity templates, and a setup script that installs everything on a fresh Mac.

## What Alfie Is

Alfie is Alfred Adler (philosopher/psychologist) transformed into a magical computer AI. It's a layer on top of Claude Code that adds:

- **31 skills** — specialized slash commands for brainstorming, planning, building, shipping, and Notion integration
- **Identity** — Alfie has a voice, philosophy, and persistent memory; not a generic assistant
- **Dev flow** — a structured methodology: `/brainstorm → /plan → /build → /review → /ship`
- **gstack integration** — Alfie installs [gstack](https://github.com/garrytan/gstack) as a dependency for the browser, compound-engineering skills, and QA tooling

## Install

```bash
git clone https://github.com/YOUR_ORG/alfie-os ~/Code/os-structure
cd ~/Code/os-structure
./setup
```

The setup script will:
1. Check Claude Code is installed (`~/.claude/` exists)
2. Install gstack if not present
3. Symlink all 31 Alfie skills into `~/.claude/skills/`
4. Copy memory templates to `~/.claude/memory/` (only on first install)
5. Generate `~/.claude/CLAUDE.md` from the template (prompts for your name, email, ventures)

After install, open Claude Code and type `/brainstorm` — you're running Alfie.

## Skills

All 31 Alfie-native skills live in `skills/`. Each is a `SKILL.md` file that Claude Code loads as a slash command.

| Category | Skills |
|----------|--------|
| **Dev flow** | brainstorm, plan, build, work, orchestrate, standup, done, codify, verify, receive-review, investigate |
| **Notion** | notion-brain, notion-read, notion-do, create-page, create-database-row, create-task, database-query, find, search, publish, link-pages, sync-docs |
| **Projects** | project-setup, weekly |
| **Design** | canvas-design, frontend-design, implement-design, framer, framer-code-components, theme-factory |
| **Research** | invest |

Skills marked `tier: personal` in their frontmatter require personal config (Notion workspace IDs, API credentials) to function. See `skills/<name>/SKILL.md` for requirements.

## gstack Dependency

Alfie depends on [gstack](https://github.com/garrytan/gstack) for:
- The compound-engineering plugin (CE brainstorm/plan engines)
- Browser-based QA (`/qa`, `/canary`)
- Code review and ship skills

gstack is installed as a dependency by `./setup` — Alfie does not fork or vendor it. To update gstack: `cd ~/.claude/skills/gstack && git pull && ./setup --quiet`.

See `docs/CURATION.md` for how Alfie monitors gstack updates and decides what to incorporate.

## Structure

```
os-structure/
├── setup                   # Install script
├── CLAUDE.md.tmpl          # Parameterized identity template
├── skills/                 # All 31 Alfie-native skills
├── memory/                 # Identity templates (copied on first install)
└── docs/
    ├── CURATION.md         # How Alfie monitors gstack updates
    ├── brainstorms/        # Design specs
    └── plans/              # Implementation plans
```

## Curation Practice

Alfie is not a fork of gstack — it's a separate product that monitors gstack as a reference. See `docs/CURATION.md` for the standing practice of reviewing gstack updates and deciding what to incorporate.

## Working from iPad / Remote Sessions

Alfie runs in two surfaces:

- **Mac (full)** — local Claude Code with Warp, gstack browser, Preview-flashed Alfie images. All 10 dev-flow steps available.
- **Remote (cloud)** — Claude Code on the web from iPad, browser, or anywhere. ASCII Alfie fallback. Steps that need a local browser are deferred.

The template auto-detects which one you're in (`$CLAUDECODE_REMOTE` set OR no `sips` on PATH → remote) and adjusts.

### What works where

| Step | Mac | Remote |
|------|-----|--------|
| `/brainstorm` | ✓ | ✓ |
| `/plan` | ✓ | ✓ |
| `/autoplan` | ✓ | ✓ |
| `/build → /work` | ✓ | ✓ |
| `/build → /orchestrate` | ✓ | falls back to `/work` |
| `/qa` | ✓ | defer (needs gstack browser) |
| `/review` | ✓ | ✓ |
| `/ship` | ✓ | ✓ |
| `/canary` | ✓ | defer (needs gstack browser) |
| `/codify` | ✓ | ✓ |
| `/done` | ✓ | ✓ |
| Alfie character | image flash | ASCII frames |

### Setting up an iPad

1. **Claude Code on the web** at claude.ai/code — Safari → Share → Add to Home Screen for an app-like surface.
2. **Bluetooth keyboard.** Slash commands aren't pleasant on the on-screen keyboard.
3. **GitHub mobile app** for PR review and merge alongside `/ship`.
4. **Optional: Claude Dispatch** (iPhone, Max/Pro) to drive your home Mac for `/qa` and `/canary` while you're remote.

### Before a trip

- Draft any `/brainstorm` and `/plan` work in advance so remote sessions can run `/build → /work` from approved plans.
- Leave the home Mac powered on with `caffeinate` running if you want Dispatch as a fallback.
- Pin Claude Code on the web to the iPad home screen and verify GitHub mobile is signed in.
