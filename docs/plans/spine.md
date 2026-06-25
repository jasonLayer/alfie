# Plan: SPINE — DO + Daily Log

**Goal:** Stand up the minimum end-to-end loop so I can capture into JR_DO_DB and run "build today's log" to produce a populated JR_Log_DB page.

**Definition of Done:**
- Repo exists at `~/Code/do-list-system`, pushed to GitHub as private repo `do-list-system`.
- `setup` script symlinks both skills into `~/.claude/skills-ext/`.
- do-capture skill creates DO rows with correct source.
- daily-log skill, on "build today's log", produces (or refreshes) a JR_Log_DB page with all three sections populated.
- One-line wire-in present in `~/.claude/CLAUDE.md`.
- Memory file written with all Notion IDs and rules.
- Test pass: a Manual or Alfie DO row appears in "Captured Today" on today's log page.

---

## Tasks

### 1. Scaffold repo (~/Code/do-list-system)
Create directory tree, init git, write `.gitignore`, write `README.md` (one-paragraph what-and-why).

### 2. Write design spec (`docs/specs/do-daily-log.md`)
Already drafted — confirm it captures philosophy, schema, rollup rules, source list.

### 3. Write all plans
`spine.md` (this file), `limb-1-alfie-source.md`, `limb-2-apple-reminders.md`, `limb-3-sticky-photo.md`, `limb-4-ios-shortcut.md`, `limb-5-apple-notes.md`, `automation-nightly-rollup.md`.

### 4. Write `skills-ext/do-capture/SKILL.md`
Define `create_do(name, source, context=None)` and `mark_done(item)`. Include Notion API call shape and the canonical source select values.

### 5. Write `skills-ext/daily-log/SKILL.md`
Define "build today's log" trigger, date resolution, page-find-or-create logic, the three-section write, and the read-only invariant.

### 6. Write `memory/jr-do-system.md`
All IDs, schemas, source list, rollup rules, architecture, principles. This is what Alfie reads at session start to know how to operate.

### 7. Write `setup` script
Idempotent. Detect existing symlinks, replace dangling ones, leave correct ones alone. Print what it did.

### 8. Notion follow-ups (via notion-personal MCP)
- Move JR_DO_DB into JR HQ → "Admin (Do Not Edit)".
- Move JR_Log_DB into JR HQ → "Admin (Do Not Edit)".
- Add Nav shortcuts "DO" and "Daily Log" in JR HQ Nav column (linked views).
- Add JR_Log_DB page template with three-section body.
- Rename old JR_DailyLog_DB to "JR_DailyLog_DB (Archive)".

### 9. Wire into `~/.claude/CLAUDE.md`
Single line at appropriate spot: "JR personal DO + Daily Log system: see `~/Code/do-list-system/memory/jr-do-system.md`. Skills do-capture and daily-log are always-on for me."

### 10. GitHub repo
- Display name: "Jason Reichl's Do List System"
- Slug: `do-list-system`
- Private
- Push initial commit
- `gh repo edit` to set display name if needed

### 11. Spine completion test
- Create a test DO row via Notion MCP: `Name="Test capture from spine completion"`, `Source=Manual`.
- Run "build today's log".
- Read today's JR_Log_DB page.
- Confirm: page exists, three sections present, test row appears under Captured Today, Creative Father section populated (or labeled empty if no quote today).

---

## Risk / Open Questions

- **JR_Log_DB page template via MCP:** the Notion API supports DB templates via the data-source `templates` endpoint. If the MCP exposes it, use it. If not, document the manual setup steps and have the daily-log skill create the structure on the fly when the page is new.
- **"Admin (Do Not Edit)" section:** if this section doesn't already exist on the JR HQ parent page, the move task should create it (as a sub-page or toggle) first.
- **Time zone:** all "today" resolution must use America/Los_Angeles.
