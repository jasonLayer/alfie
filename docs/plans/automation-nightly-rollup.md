# Plan: Automation — nightly rollup

**Goal:** Today's JR_Log_DB page exists, populated with all three sections, by the time I wake up. No "build today's log" required to get the morning view.

**Pre-req:** Spine shipped. daily-log skill stable.

**Definition of Done:**
- A scheduled job runs at 11:55 PM America/Los_Angeles each night and again at 5:30 AM the next morning.
- 11:55 PM run finalizes the *outgoing* day (captures last-minute completions).
- 5:30 AM run pre-builds the *new* day so the morning standup has a ready surface.
- Failure to run is surfaced (notification or log line in a known place).

---

## Tasks

### 1. Pick the scheduler
Options:
- `launchd` LaunchAgent on the Mac (must be awake).
- Cron on a small VPS or Cloudflare Worker scheduled trigger.
- Existing Claude Code `/schedule` cron facility (if available and reliable for this).

Default: Cloudflare Worker scheduled trigger, calling the same worker used by LIMB 4 with an extra path `/build-log` that invokes the rollup logic server-side.

This means the rollup logic needs to be portable enough to run outside of a Claude session. Two options:
- **a)** A standalone Node/TS port of the daily-log skill that calls Notion APIs directly.
- **b)** Trigger a Claude Agent SDK invocation that runs the skill.

Default: (a) — a TS port living in this repo at `bin/rollup.ts`, deployed alongside the worker.

### 2. Port the daily-log logic
Translate the skill's three-section logic into TypeScript:
- Resolve target date (LA time).
- Find-or-create the JR_Log_DB page.
- Query the three source DBs.
- Render markdown blocks; PATCH the page body sections.

### 3. Idempotency
Run can fire multiple times the same day; outcome should be identical. The skill already handles this — confirm the TS port preserves it.

### 4. Observability
- Worker logs to its built-in log surface.
- On failure, send a single-line email or push notification to me (Cloudflare → my address).
- Don't retry more than 3 times to avoid runaway runs.

### 5. Disable on weekends? — No
Even on weekends, the Daily Log is useful. Run every day.

### 6. Manual override
The "build today's log" skill remains the canonical interactive trigger. The nightly job is just there so the morning view is ready when I sit down.

---

## Risk / Open Questions

- **Drift between skill and TS port:** they must produce identical output. Plan to share a fixture test (sample input → expected markdown) and run it against both.
- **Notion rate limits:** at 5:30 AM, with three DB queries, we're fine. Document the rate budget so future limbs don't pile on.
- **Time-zone bugs:** verify around DST transitions explicitly.
