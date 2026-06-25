# Plan: LIMB 2 — Apple Reminders two-way sync

**Goal:** Items I capture in Apple Reminders show up in JR_DO_DB with `Source=Apple Reminders`. Items I complete in either system flip the other to completed. No manual export/import.

**Pre-req:** Spine shipped. Decision on push frequency (poll every N minutes vs realtime).

**Definition of Done:**
- A new Reminder in a designated list ("DO" or similar) appears as a JR_DO_DB row within the configured sync window.
- Marking the Reminder complete sets `Completed = true` and `Date Completed = today` in DO.
- Marking the DO row complete from any other source closes the corresponding Reminder.
- Deleted Reminders are not auto-deleted from DO (capture is permanent; processing is manual).

---

## Tasks

### 1. Designate the bridge list
Use a dedicated Reminders list named "DO" so we don't sync the user's existing personal/work lists. New items go there; other lists are ignored.

### 2. Sync mechanism
Options:
- **EventKit + macOS daemon (preferred):** native Reminders access, can run as a launchd job, polls every 30s or subscribes to change notifications. Built in Swift or via JXA.
- **Shortcuts automation:** "When item added to DO list, run shortcut that calls webhook." Simpler but less reliable across devices.
- **iCloud CalDAV:** technically possible but messy.

Default plan: launchd-managed Swift binary in this repo at `bin/reminders-sync`. Logs to `~/Library/Logs/jr-do-reminders-sync.log`.

### 3. ID mapping
Keep a local SQLite (or JSON) file mapping `reminder_uuid → notion_page_id`. Stored in `~/.local/share/jr-do-system/mappings.db`. Reads on startup, writes on each new sync.

### 4. Push directions
- **Reminders → DO:** new items create JR_DO_DB rows with the reminder title as `Name`, `Source=Apple Reminders`. The reminder's notes field becomes the page body. Completed reminders mark the row complete.
- **DO → Reminders:** when a DO row with a mapped reminder is marked complete, close the reminder. (Do not push DO → Reminders for newly-created rows; only the completion direction.)

### 5. Conflict handling
- Both completed on the same day: both stay completed, no action.
- One completed, the other not: latest wins; the unfinished side gets marked complete.
- Deleted on one side: ignore; the orphan persists.

### 6. Privacy
The sync daemon needs Reminders access (System Settings → Privacy → Reminders). Document the prompt and what to allow.

### 7. Onboarding
A `make sync-install` target installs the launchd plist and starts the daemon. `make sync-uninstall` removes it.

---

## Risk / Open Questions

- **EventKit on macOS:** the API is stable but writes from a background process need careful permission handling.
- **Time zones:** Reminders stores completion timestamps in UTC; convert to America/Los_Angeles when setting `Date Completed`.
- **Cross-device latency:** Reminders syncs through iCloud, which can lag. The daemon should retry if a recently-added item lacks a stable UUID.
