# Plan: LIMB 4 — iOS Shortcut single-tap capture

**Goal:** A Shortcut on the iOS Home Screen (and via Siri) takes a single utterance or text entry and creates a JR_DO_DB row with `Source=Shortcut` in under three seconds.

**Pre-req:** Spine shipped. A small HTTP endpoint or callable Notion proxy that the Shortcut can hit.

**Definition of Done:**
- Tapping the Shortcut from the lock screen or saying "Hey Siri, DO this" prompts for input.
- Input becomes a new DO row within ~2 seconds.
- The Shortcut works offline-deferred: queues locally if no network, fires when reconnected.
- The Shortcut icon and name are dialed in (something fast to find — "DO" or a single-emoji icon).

---

## Tasks

### 1. Choose the endpoint
Two flavors:
- **Notion API direct from the Shortcut:** Shortcuts can call HTTP with a header containing a Notion integration token. Simplest, but exposes the token in the Shortcut. OK if iCloud Shortcut sync is trusted.
- **Tiny proxy (Cloudflare Worker / Vercel function):** the Shortcut calls a personal endpoint, which holds the Notion token. Safer and lets us add validation/logging later.

Default: Cloudflare Worker named `jr-do-capture` deployed under a personal domain.

### 2. Worker implementation
- POST `/capture` with `{ name, source, context? }`.
- Auth via a shared secret header (rotated yearly).
- Calls Notion `pages.create` against JR_DO_DB.
- Returns `{ ok: true, page_id, url }`.

### 3. Build the Shortcut
Steps:
1. Dictate text (or accept text input).
2. POST to the worker with `name=text`, `source="Shortcut"`.
3. On success: short haptic + notification "Captured."
4. On failure: queue payload in a local file (Shortcuts' "Saved" storage) and retry on next run.

### 4. Voice variants
Configure Siri shortcuts:
- "DO this" — opens with dictation.
- "Capture {phrase}" — direct capture, skips the dictation UI.

### 5. Lock screen widget
Add the Shortcut as a lock-screen button (iOS 16+). Single tap → capture flow.

### 6. Test plan
- Cold start (phone in pocket): tap → capture → verify row in <3s.
- Airplane mode: tap → capture → reconnect → verify row arrives.
- Siri voice: "Hey Siri, DO this" → say something → verify row + accurate transcription.

---

## Risk / Open Questions

- **Shortcut sync flakiness:** Shortcuts.app has a habit of breaking on iOS upgrades. Document the full setup in this plan file so reconstruction is trivial.
- **Secret rotation:** the worker secret lives in the Shortcut. Rotation requires touching every device with the Shortcut installed. Annual is fine.
- **Source attribution:** if the same Shortcut is reused on iPad, the source is still `Shortcut`. Don't split by device unless I later want device-level analytics.
