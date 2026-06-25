# Plan: LIMB 3 — Sticky-note photo capture

**Goal:** Take a photo of a physical sticky note (or whiteboard scrap), drop it into a known location, and have it parsed into JR_DO_DB rows with `Source=Sticky`.

**Pre-req:** Spine shipped. Vision-capable model available (Claude with images is fine).

**Definition of Done:**
- Photos placed in `~/Pictures/Sticky Capture/` (or shared from iPhone) are processed within a configured window.
- Each distinct line/item on the sticky becomes its own JR_DO_DB row.
- The original photo is stored as a page block (or linked from cloud storage) in the body of the first row, with a back-link from subsequent rows.
- Processed photos are moved to `~/Pictures/Sticky Capture/processed/` with a timestamp prefix.

---

## Tasks

### 1. Inbox folder
Create `~/Pictures/Sticky Capture/` as the watched inbox. Document a Shortcut (or AirDrop target) that drops photos from iPhone directly here.

### 2. Watcher
A simple Python or Swift script using `fswatch` (already on Jason's Mac) or `launchd WatchPaths`. On new file, run the vision parser.

### 3. Vision parsing
Send the image to Claude with a prompt like:
> Extract a bulleted list of every distinct to-do item or action on this sticky note. Return as a plain JSON array of short verb-phrase strings. Ignore decorations, doodles, dates, and signatures unless they're part of an action.

Validate the response: ≤ 20 items, each ≤ 120 chars. Reject and queue for manual review if it looks off.

### 4. Create DO rows
For each parsed item, call `create_do(name=item, source="Sticky", context=…)`. The first row's body contains the photo; subsequent rows link to the first.

### 5. Photo storage
Two options:
- Upload to a private Notion file block (simplest if the MCP supports file uploads).
- Store locally and link by `file://` URL (works for personal use but breaks if the path changes).

Default: upload to Notion as a file attachment block. If the MCP doesn't support uploads, fall back to a private S3/R2 bucket with signed URLs.

### 6. Move processed
Once all rows are created, move the photo to `processed/YYYY-MM-DD-HHMMSS-<original-name>`.

### 7. Error handling
On parse failure, move the photo to `failed/` with a `.error.txt` next to it explaining why. Notify via terminal/macOS notification.

---

## Risk / Open Questions

- **Handwriting quality:** if my handwriting is bad, the model will hallucinate. Add a confidence check: if any item starts with a generic verb but the rest is gibberish, flag it.
- **Multi-sticky photos:** a single photo with multiple stickies should be handled — the prompt covers it but test with a few real examples.
- **Privacy:** photos may capture sensitive context. The watched folder should never auto-sync to public storage.
