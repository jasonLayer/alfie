# Plan: LIMB 5 — Apple Notes ingestion

**Goal:** Items in a designated Apple Notes folder (or notes with a designated tag) are ingested into JR_DO_DB with `Source=Apple Notes`.

**Pre-req:** Spine shipped. Decision on whether ingestion is line-by-line or note-as-row.

**Definition of Done:**
- New notes (or new bulleted lines within a tracked note) in the configured folder appear as DO rows.
- Already-ingested items are not re-ingested.
- The original note is referenced in the DO row body for context.

---

## Tasks

### 1. Designate the source
Default: an Apple Notes folder named "DO Inbox." Anything in that folder is fair game.

Alternative: notes anywhere with the tag `#do`. More flexible but harder to scope.

### 2. Access mechanism
Apple Notes has no public API. Options:
- **AppleScript / JXA against Notes.app:** works on Mac only; the script must run while Notes is reachable.
- **iCloud Notes export (.notes):** not a public format.
- **Manual sync via a Shortcut:** the user runs a shortcut that reads the folder and POSTs to the same Worker as LIMB 4.

Default: AppleScript-based watcher running on the Mac under launchd, polling every 10 minutes.

### 3. Ingestion rules
For each note in the inbox folder:
- If the note has bulleted lines, each bullet becomes a DO row.
- If it's a single paragraph, the note title (or first 80 chars) becomes the row name; the body becomes the context.

Maintain an ingestion-log file (`~/.local/share/jr-do-system/apple-notes-seen.json`) mapping note ID + bullet hash → DO page ID so re-runs are idempotent.

### 4. Post-ingestion behavior
- Move the note to an "Ingested" subfolder (or strikethrough the bullets).
- Never delete from Apple Notes — the user should be the only one deleting from their own Notes app.

### 5. Test
- Drop a note with 3 bullets in DO Inbox → confirm 3 DO rows with Source=Apple Notes.
- Add a 4th bullet → confirm only the new bullet creates a row.
- Edit an existing bullet's text → no new row, existing row untouched (we don't follow updates).

---

## Risk / Open Questions

- **AppleScript fragility:** Notes.app's AppleScript surface changes between macOS versions. Pin a tested version of macOS and document quirks.
- **Cross-device sync:** if I edit a note on iPhone but the Mac is asleep, ingestion is delayed until the Mac wakes. Acceptable for a personal system.
- **Overlap with LIMB 4:** if the iOS Shortcut already captures everything, this limb may not be worth building. Defer until I find I'm putting things in Notes that should be in DO.
