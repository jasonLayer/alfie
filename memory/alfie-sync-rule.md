---
name: alfie-sync-rule
description: Two-way sync rule — changes to Alfie in Claude Code must propagate to Notion, and Notion Alfie pages should be checked for drift
type: feedback
---

## Rule: Alfie is a Two-Way System

Alfie's configuration lives in two places that must stay in sync:
- **Claude Code:** `~/.claude/CLAUDE.md`, `~/.claude/memory/alfie-*.md`, `~/.claude/projects/-Users-*/memory/who-is-jason.md`
- **Notion:** Alfie Core, Voice & Style, Workflows, Adler Skills, Chief of Staff dossier

### When Jason changes Alfie in Claude Code (conversation)

If Jason gives feedback that changes Alfie's identity, voice, priorities, behavior, preferences, or watch list:

1. Update the relevant local memory file (`alfie-identity.md`, `alfie-voice.md`, `alfie-chief-of-staff.md`, or `who-is-jason.md`)
2. Determine which Notion page is affected:
   - Identity/priorities → [Alfie Core](https://www.notion.so/45dd0b00d32348ec8db64c6f852b7af2) (esp. Live Preferences & Overrides)
   - Voice/communication → [Alfie Voice & Style](https://www.notion.so/d993a38316674c998e6c6a4fed938030)
   - Workflows → [Alfie Workflows](https://www.notion.so/fb55badf28184f8290a7215c6ff39699)
   - Coaching/psychology → [Alfie's Adler Skills](https://www.notion.so/28a0d32a2816807cb83bd98a63665579)
   - About Jason → [Chief of Staff](https://www.notion.so/9ae6fe1de3624d4182a57b420ba4d542)
   - Capabilities summary → [Reference doc](https://www.notion.so/589cc4f80dec4a2691e400323ec2cd50)
3. Update the Notion page to match
4. Confirm to Jason: "Updated locally and in Notion."

### Periodic Notion check

At the start of longer sessions (or when Jason asks), check the Alfie Notion pages for changes that haven't been synced to local memory:

1. Fetch [Alfie Core](https://www.notion.so/45dd0b00d32348ec8db64c6f852b7af2) — check Live Preferences & Overrides section
2. Fetch [Chief of Staff](https://www.notion.so/9ae6fe1de3624d4182a57b420ba4d542) — check for updated priorities or context
3. If anything has changed, update the corresponding local memory file and mention it

**Why:** Alfie's Notion pages may be updated directly by Notion AI or by Jason outside of Claude Code. Without syncing, the two versions drift apart and Alfie behaves inconsistently across surfaces.

**How to apply:** Any time a conversation includes feedback about Alfie's behavior, voice, priorities, or knowledge — treat it as a change to both systems, not just the current conversation.

### CCStatusLine Config Sync

CCStatusLine config is tracked in the alfie-os repo:
- **Canonical file:** `~/.claude/config/ccstatusline/settings.json`
- **Live location:** `~/.config/ccstatusline/settings.json` → symlinked to the canonical file
- **Repo:** `https://github.com/jasonLayer/alfie-os` (branch: `main`)

If Alfie updates `~/.claude/config/ccstatusline/settings.json` for any reason (new widget, layout change, etc.):

1. Stage and commit the change: `git -C ~/.claude add config/ccstatusline/settings.json && git -C ~/.claude commit -m "..." && git -C ~/.claude push origin main`
2. Tell Jason: "Updated CCStatusLine config and pushed to alfie-os. Run `git -C ~/.claude pull` on your other machines to sync."

If Jason changes the config on another machine and pushes, Alfie should pull: `git -C ~/.claude pull`

### Project Registry Sync

The project registry also lives in both places:
- **Notion:** [Reference: Project Registry](https://www.notion.so/3250d32a2816815dafa0dac4620a3f89) (under Alfie OS app)
- **Local:** `~/.claude/memory/project-registry.md`

**Notion is the primary source of truth.** Jason edits there to control auto-publish routing. At the start of sessions in `~/Code/os-structure`, fetch the Notion page and sync to local. If Alfie adds a project (via `/project-setup`), update both.
