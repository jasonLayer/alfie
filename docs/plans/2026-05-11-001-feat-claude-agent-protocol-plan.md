---
id: 2026-05-11-001
title: "feat: Build Claude Agent Protocol — Personal GTM System"
type: feat
status: active
origin: docs/specs/claude-agent-protocol-design.md
created: 2026-05-11
depth: deep
---

# feat: Build Claude Agent Protocol — Personal GTM System

**Origin doc:** `docs/specs/claude-agent-protocol-design.md`  
**Project root (to be created):** `~/Code/claude-agent-protocol/`  
**Week 1 target:** Researcher agent functional on TrustLayer context  

---

## Summary

Build a file-system-native, three-agent GTM automation system that runs inside a standalone Claude Code project. Agents coordinate via flat markdown files — no server, no event bus, no database. A single `ACTIVE_CONTEXT` variable in `CLAUDE.md` switches the system between TrustLayer sales mode and Jason's personal project outreach mode.

The plan follows the Peslar Claude Agent Protocol pattern: Claude as operator, not chatbot. Sequenced 4-week activation: Researcher (Week 1) → Writer + Voice DNA (Week 2) → Reply Triage + First Sends (Week 3) → Autonomous daily loop (Week 4).

All agent files are markdown documents read as context at session start. No runtime server. Jason approves queue items daily; nothing sends without explicit sign-off.

---

## Problem Frame

Manual GTM outreach is expensive and inconsistent. Jason runs two separate sales/outreach contexts (TrustLayer AE role and personal venture/consulting work) with no systematic research, drafting, or reply-tracking layer. The Peslar framework provides a proven pattern: file-system-as-message-bus, three narrowly scoped agents, voice DNA as style guard, ICP scoring to prioritize effort.

---

## Scope Boundaries

### In scope
- Project directory scaffold at `~/Code/claude-agent-protocol/`
- Dual-context `CLAUDE.md` with `ACTIVE_CONTEXT` and `CURRENT_PROJECT` variables
- TrustLayer context file with full ICP, voice guidance, and tool routing
- Jason base context + 4 project ICP files (MajorGTM, POAM, 8thday, RevOps/AI consulting)
- Researcher agent: prompt + Single-Command Audit template + dossier schema
- Writer agent: prompt + queue format + voice DNA check protocol
- Reply Triage agent: prompt + classification rules + CRM update protocol
- Voice DNA templates (empty, ready for Week 2 extraction)
- Schema validation scripts for dossier and queue file formats
- Operator README with session startup checklist and MCP setup guide
- `.gitignore` excluding `crm/` and `queue/` (personal contact data)

### Deferred to Follow-Up Work
- HubSpot CRM sync (open question — flat files may be sufficient long-term)
- MajorGTM integration (future: shared conventions or shared module, TBD)
- Automated scheduling / cron-based triage (graduate after Week 4 proves the loop)
- Zevari sends (Week 3 activation — not part of this scaffold build)

### Outside this product's identity
- Backend server, hosted API, or database layer
- SaaS product or multi-user system
- Replacement for HubSpot / full CRM

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Project isolation | Standalone `~/Code/claude-agent-protocol/` with own `CLAUDE.md` | Operator brief only takes effect when Claude Code runs in this directory; avoids polluting Alfie's `~/Code` CLAUDE.md |
| Context switching | `ACTIVE_CONTEXT` variable at top of `CLAUDE.md`, loaded at session start | Single file to edit; Claude reads it before any user message |
| Agent files | Markdown system prompts in `agents/` | Read as context at session start; no special tooling required |
| Message bus | Flat `.md` files in `crm/` and `queue/` subdirectories per context | No infrastructure, inspectable by Jason, git-diffable |
| ICP files | One `.md` per project under `contexts/jason/` | Loaded conditionally based on `CURRENT_PROJECT`; easy to add new projects |
| Voice DNA check | Inline instruction in Writer system prompt, not a separate tool | Keeps the Writer self-contained; no external validation step needed |
| Queue approval | Jason edits `Status: draft → approved` manually before sending | No accidental sends; Jason stays in control at all times |
| Schema validation | Bash scripts in `scripts/` | Lightweight, no dependencies, validate output format without running agents |
| Personal data | `crm/` and `queue/` in `.gitignore` | Contact data should not be in version control |

---

## Output Structure

```
claude-agent-protocol/
├── .gitignore
├── README.md
├── CLAUDE.md
├── contexts/
│   ├── trustlayer.md
│   └── jason/
│       ├── majorgtm.md
│       ├── poam.md
│       ├── 8thday.md
│       └── revconsulting.md
├── voice-dna/
│   ├── trustlayer.md        ← empty template, filled Week 2
│   └── jason.md             ← empty template, filled Week 2
├── agents/
│   ├── researcher.md
│   ├── writer.md
│   └── reply-triage.md
├── crm/
│   ├── trustlayer/          ← {firstname-lastname}.md per prospect
│   └── jason/
├── queue/
│   ├── trustlayer/          ← {name}-{step}.md drafts
│   └── jason/
└── scripts/
    ├── validate-dossier.sh
    └── validate-queue.sh
```

---

## Implementation Units

### U1. Project scaffold and `.gitignore`

**Goal:** Create the directory structure and root config files for the standalone project.

**Requirements:** Project must function as an isolated Claude Code session root. Personal data in `crm/` and `queue/` must not be committed.

**Dependencies:** None.

**Files:**
- `claude-agent-protocol/` (directory, new)
- `claude-agent-protocol/.gitignore`
- `claude-agent-protocol/crm/trustlayer/.gitkeep`
- `claude-agent-protocol/crm/jason/.gitkeep`
- `claude-agent-protocol/queue/trustlayer/.gitkeep`
- `claude-agent-protocol/queue/jason/.gitkeep`
- `claude-agent-protocol/voice-dna/` (directory)
- `claude-agent-protocol/scripts/` (directory)

**Approach:** Create the full directory tree. `.gitignore` must ignore `crm/` and `queue/` contents but preserve the directories themselves via `.gitkeep` files. Include standard ignores for `.DS_Store` and `*.tmp`.

**Test scenarios:**
- `.gitignore` ignores `crm/trustlayer/jane-doe.md` and `queue/trustlayer/jane-doe-1.md`
- `.gitignore` does NOT ignore `crm/trustlayer/.gitkeep` (directories stay tracked)
- Running `git status` after adding a dossier file shows it as untracked but ignored
- Directory structure matches the Output Structure tree exactly

**Verification:** `tree claude-agent-protocol/` shows the full structure. `git check-ignore -v crm/trustlayer/test.md` confirms ignore rule fires.

---

### U2. `CLAUDE.md` — Operator Brief

**Goal:** Write the project CLAUDE.md that makes Claude act as GTM Operator when running in this directory. Sets `ACTIVE_CONTEXT` and `CURRENT_PROJECT` variables and routes Claude to the correct context file.

**Requirements:** Claude must read the context file matching `ACTIVE_CONTEXT` on every session start. For `jason` mode, must also load the project-specific ICP file matching `CURRENT_PROJECT`.

**Dependencies:** U1.

**Files:**
- `claude-agent-protocol/CLAUDE.md`

**Approach:** Structure as the Peslar operator brief. Top two lines are the mutable variables Jason edits per session. Then six sections: Role, Context (→ load instruction), Tools Available (MCP list by context), Output Format, Constraints, Kill Switches.

Context loading instruction format:
```
Read contexts/{ACTIVE_CONTEXT}.md now. If ACTIVE_CONTEXT=jason, also read contexts/jason/{CURRENT_PROJECT}.md.
```

Kill Switches section must include:
- `PAUSE_OUTREACH: false` — set to `true` to halt all sends immediately
- `REVIEW_MODE: false` — set to `true` to require explicit approval on every action, not just sends

**Test scenarios:**
- Jason opens Claude Code in `claude-agent-protocol/` with `ACTIVE_CONTEXT: trustlayer` — Claude introduces itself as TrustLayer GTM Operator and references TL ICP
- Jason changes to `ACTIVE_CONTEXT: jason` and `CURRENT_PROJECT: majorgtm` — Claude references MajorGTM ICP on next session start
- `PAUSE_OUTREACH: true` causes Writer to refuse to add `Status: approved` to queue files
- Kill switch section is the last section in CLAUDE.md

**Verification:** Open a new Claude Code session in the project directory. Claude's first response references the active context ICP without being asked.

---

### U3. TrustLayer context file

**Goal:** Write the complete TrustLayer context document loaded by Claude when `ACTIVE_CONTEXT: trustlayer`.

**Requirements:** Must contain full ICP, pain signals, trigger events, target titles, voice guidance, tool routing, email identity, and CRM/queue paths. (see origin: `docs/specs/claude-agent-protocol-design.md` — TrustLayer ICP section)

**Dependencies:** U1.

**Files:**
- `claude-agent-protocol/contexts/trustlayer.md`

**Approach:** Structure as a Claude-readable briefing document, not prose. Use clear sections with explicit labels. ICP section must include scoring guidance (what makes a 9-10 vs a 3-4) so Researcher can calibrate.

Required sections:
1. Identity (who Jason is in this context, email, LinkedIn profile)
2. ICP (primary + secondary buyer, company profile, pain, trigger events)
3. ICP Scoring Rubric (criteria for 1-10 scale — specific, not vague)
4. Voice (tone adjectives, banned phrases placeholder, example sentences)
5. Tools (Zevari for LinkedIn, Gmail MCP for email, mcp__notion-trustlayer__ for lookup)
6. Paths (crm/trustlayer/, queue/trustlayer/)
7. Current Campaign (placeholder — Jason fills per sprint)

**Test scenarios:**
- ICP Scoring Rubric has explicit criteria for scores 1-3 (poor fit), 4-6 (moderate), 7-10 (strong)
- Banned phrases section exists even if initially empty (so Writer knows where to look)
- Email identity section shows `jason@trustlayer.io`
- CURRENT_CAMPAIGN section exists as a clearly labeled placeholder

**Verification:** Read the file — a new Claude session can answer "who am I targeting and why?" without asking any follow-up questions.

---

### U4. Jason context and project ICP files

**Goal:** Write the Jason base context and all four project ICP files for personal outreach mode.

**Requirements:** Jason base context must explain the CURRENT_PROJECT pattern. Each project ICP must be specific enough that Researcher can score prospects and Writer can personalize drafts without further context. (see origin: `docs/specs/claude-agent-protocol-design.md` — Jason project ICP files)

**Dependencies:** U1.

**Files:**
- `claude-agent-protocol/contexts/jason.md`
- `claude-agent-protocol/contexts/jason/majorgtm.md`
- `claude-agent-protocol/contexts/jason/poam.md`
- `claude-agent-protocol/contexts/jason/8thday.md`
- `claude-agent-protocol/contexts/jason/revconsulting.md`

**Approach:** `jason.md` is the base — Jason's identity, personal email, LinkedIn, Notion routing (mcp__notion-personal__), and instruction to load `contexts/jason/{CURRENT_PROJECT}.md`.

Each project file uses the same sections as `trustlayer.md` (Identity override, ICP, Scoring Rubric, Voice, Tools, Paths, Current Campaign). Project files override only what differs from the base.

ICP specifics per project:
- **MajorGTM**: GTM operators/founders at early-stage B2B who feel the pain of manual SDR work and tool overload. Title: Head of Growth, VP Sales, Founder. Pain trigger: scaling outbound without hiring.
- **POAM**: Bay Area curators, gallery directors, collectors, venue operators, fellow artists. Relationship-first — not commercial outreach. Hook: genuine artistic connection or shared aesthetic.
- **8thday coaching**: Executives/founders (CEO, COO, Partner-level) feeling isolation, overwork, or clarity loss. Sensitive context — never lead with "burnout." Lead with growth and potential.
- **RevOps/AI consulting**: RevOps leaders, CROs, and GTM Engineers at B2B SaaS (Series B-D) who want to automate their stack or have AI-adjacent initiatives stalled. Pain: tool sprawl, manual reporting, AI projects with no ROI.

**Test scenarios:**
- Each project file answers: "who am I targeting, what's their pain, and what's the hook?" without ambiguity
- 8thday file explicitly notes to never use the word "burnout" in outreach
- POAM file has a lower ICP-score ceiling (max 7) — this is relationship cultivation, not commercial pipeline
- RevOps file specifies Series B-D as company stage filter

**Verification:** Switch CURRENT_PROJECT to each project and ask Claude "who am I trying to reach and what's the first message hook?" — answers should be specific and distinct per project.

---

### U5. Researcher agent

**Goal:** Write the Researcher system prompt with Single-Command Audit template, dossier schema, tool usage sequence, and ICP scoring constraint.

**Requirements:** Researcher is read-only. No outreach. No queue writes. Output is always a dossier at `crm/{context}/{firstname-lastname}.md`. Tool usage follows a defined fallback order. (see origin: `docs/specs/claude-agent-protocol-design.md` — Agent 1: Researcher)

**Dependencies:** U3, U4.

**Files:**
- `claude-agent-protocol/agents/researcher.md`

**Approach:** The file is a system prompt Claude reads at session start when research mode is active. Structure:

1. **Role declaration**: "You are the Researcher. You gather intelligence on prospects. You do not send messages, write queue files, or contact anyone."
2. **Tool usage sequence**: Zevari (LinkedIn profile read) → Serper (web search, last 30 days) → Jina (company website, press) → Playwright (fallback for paywalled pages). Stop when dossier has enough to score confidently.
3. **Single-Command Audit template**: The structured prompt Jason pastes to trigger a full research run (from Peslar article, adapted for dual-context).
4. **Dossier schema**: Exact markdown format with all 9 required fields. ICP Score must be a number with one-sentence rationale in parentheses.
5. **Scoring constraint**: ICP Score must use the rubric in the active context file. Never give a 9-10 without citing a specific trigger event signal.
6. **Save instruction**: Always save as `crm/{ACTIVE_CONTEXT}/{firstname-lastname}.md`. Overwrite if file already exists, preserve original Research date.

**Execution note:** TDD-first for schema — run `scripts/validate-dossier.sh` against the output after the first research run.

**Test scenarios:**
- Researcher run on a known public LinkedIn URL produces a dossier file at correct path
- ICP Score field contains a number (1-10) and a parenthetical rationale
- All 9 dossier fields are present and non-empty
- Researcher does NOT create any files in `queue/`
- Dossier overwrites correctly when re-run on same prospect (Research date updates)
- ICP Score 9-10 requires citing a trigger event — a score of 9 without a trigger event is a failure

**Verification:** After running Researcher on 3 prospects, `scripts/validate-dossier.sh crm/trustlayer/` passes with zero errors.

---

### U6. Writer agent

**Goal:** Write the Writer system prompt with queue file schema, voice DNA check protocol, and sequence step logic.

**Requirements:** Writer reads dossiers, writes queue files. Never sends. Every draft includes a Voice DNA check result. Status must always start as `draft`. (see origin: `docs/specs/claude-agent-protocol-design.md` — Agent 2: Writer)

**Dependencies:** U5.

**Files:**
- `claude-agent-protocol/agents/writer.md`

**Approach:** System prompt structure:

1. **Role declaration**: "You are the Writer. You turn dossiers into outreach drafts. You never send. You only write to queue/."
2. **Queue file schema**: Exact format with all 7 fields. Status always starts as `draft`. Never set `approved` or `sent` — Jason does that.
3. **Voice DNA check protocol**: Before finalizing the body, check every sentence against `voice-dna/{ACTIVE_CONTEXT}.md`. If the file is empty, write `Voice DNA check: NOT YET EXTRACTED — review manually` and proceed. Once populated, any banned phrase triggers a rewrite, not just a flag.
4. **Sequence step logic**: Step 1 = connection note (LinkedIn) or cold intro (email), 160 chars max. Step 2 = follow-up referencing step 1 (only if step 1 exists in queue). Step 3 = final attempt, explicit ask.
5. **Channel selection**: Default to LinkedIn Step 1. Add email Step 1 only if dossier shows email is confirmed AND LinkedIn is cold/unresponded.
6. **Hook sourcing**: First sentence of body must use the `Hook` field from the dossier verbatim or closely adapted.

**Test scenarios:**
- Writer output for any dossier always has Status: draft (never approved)
- Step 1 LinkedIn message body is ≤160 characters
- Voice DNA check field is never empty — it shows either "passed", "flagged — [reason]", or "NOT YET EXTRACTED"
- Hook from dossier appears in or clearly shapes the first sentence of the body
- Writer does NOT modify any files in `crm/`
- Step 2 file references Step 1 status (does not generate Step 2 if Step 1 hasn't been sent)

**Verification:** After running Writer on 3 dossiers, `scripts/validate-queue.sh queue/trustlayer/` passes. Manual review confirms hook usage in each draft.

---

### U7. Reply Triage agent

**Goal:** Write the Reply Triage system prompt with inbox scan procedure, classification rules, CRM update protocol, and calendar booking flow.

**Requirements:** Reply Triage classifies replies and updates CRM. UNSUBSCRIBE is immutable — once flagged, never contact again. INTERESTED fast-paths to calendar. (see origin: `docs/specs/claude-agent-protocol-design.md` — Agent 3: Reply Triage)

**Dependencies:** U5, U6.

**Files:**
- `claude-agent-protocol/agents/reply-triage.md`

**Approach:** System prompt structure:

1. **Role declaration**: "You are Reply Triage. You scan the inbox, classify each reply, and update the CRM. You do not send messages."
2. **Inbox scan procedure**: Check Gmail MCP for threads matching prospect names/emails in the active CRM directory. Check Zevari LinkedIn inbox for connection request responses. Scan in this order.
3. **Classification rules** (with explicit CRM update instruction per class):
   - `INTERESTED` — Add `Status: INTERESTED` + `Follow-up: CALENDAR` to dossier. Suggest 3 calendar slots using Google Calendar MCP. Write a `{name}-reply.md` in queue with a suggested calendar-booking response.
   - `NOT NOW` — Add `Status: NOT NOW` + `Follow-up: [date 60 days out]` to dossier.
   - `WRONG PERSON` — Add `Status: WRONG PERSON` + note any referral name mentioned.
   - `UNSUBSCRIBE` — Add `Status: UNSUBSCRIBE` to dossier. Add prospect to `queue/{context}/unsubscribe-list.md`. Never contact again under any circumstances.
   - `NO REPLY` — No CRM update needed; sequence step logic handles follow-ups.
4. **UNSUBSCRIBE guard**: Before any action on a prospect, check `unsubscribe-list.md`. If found, refuse to draft or suggest any outreach.

**Test scenarios:**
- A reply with "not interested" → classified as UNSUBSCRIBE, dossier updated, name added to unsubscribe list
- A reply with "would love to chat" → classified as INTERESTED, 3 calendar slots suggested, reply draft written to queue
- A reply with "reach out in Q4" → classified as NOT NOW, follow-up date set to ~90 days out
- UNSUBSCRIBE classification cannot be overwritten — running Triage again does not reclassify
- Prospect on unsubscribe list → Triage refuses to write any queue file for them, even if asked
- Reply Triage scans BOTH Gmail and Zevari LinkedIn inbox, not just one

**Verification:** Run Reply Triage on a test inbox with 3 seeded replies (one INTERESTED, one NOT NOW, one UNSUBSCRIBE). All three CRM files update correctly, unsubscribe list is populated, calendar suggestions appear for INTERESTED.

---

### U8. Voice DNA templates

**Goal:** Create empty voice DNA template files that Researcher and Writer can reference before Week 2 extraction, and a guide for the extraction process.

**Requirements:** Writer reads these files on every draft. Empty template must be machine-readable (no errors) and include a clear "NOT EXTRACTED YET" signal. Extraction guide must be self-contained — Jason can run it without referring back to this plan.

**Dependencies:** U1.

**Files:**
- `claude-agent-protocol/voice-dna/trustlayer.md`
- `claude-agent-protocol/voice-dna/jason.md`
- `claude-agent-protocol/docs/voice-dna-extraction-guide.md`

**Approach:** Template structure (both files identical structure, different placeholders):
```
# Voice DNA — {Context}
## Status: NOT EXTRACTED — run extraction before Week 2
## Tone adjectives: [FILL]
## Sentence length: [FILL] avg words, range [FILL]-[FILL]
## Opening styles: [FILL]
## Phrases I use (3+ times): [FILL]
## Banned phrases: [FILL]
## Punctuation style: [FILL]
## How I end messages: [FILL]
```

Extraction guide includes: the 20-message collection checklist, the exact extraction prompt to paste into Claude, how to review and edit the output, and when to consider the DNA "ready."

**Test scenarios:**
- Writer encountering an empty template outputs "NOT YET EXTRACTED — review manually" in the Voice DNA check field (not an error)
- Template has all 8 required sections even when empty
- Extraction guide extraction prompt is a copy-paste-ready single block, not fragmented instructions

**Verification:** Open voice-dna/trustlayer.md — all 8 sections present. Run Writer before Week 2 extraction — Voice DNA check field reads "NOT YET EXTRACTED — review manually", draft still produces a valid queue file.

---

### U9. Schema validation scripts

**Goal:** Write bash validation scripts that verify dossier and queue files match their required schemas.

**Requirements:** Scripts must run without any dependencies beyond standard macOS bash. Exit 0 on pass, exit 1 with specific error messages on failure. Designed to run after each Researcher or Writer session.

**Dependencies:** U5, U6.

**Files:**
- `claude-agent-protocol/scripts/validate-dossier.sh`
- `claude-agent-protocol/scripts/validate-queue.sh`

**Approach:**

`validate-dossier.sh {directory}`:
- Accepts a directory path (e.g., `crm/trustlayer/`)
- For each `.md` file: checks that all 9 required field headers exist
- Checks ICP Score field contains a numeric value 1-10
- Checks Research date matches `YYYY-MM-DD` format
- Prints PASS/FAIL per file, exits 1 if any file fails

`validate-queue.sh {directory}`:
- Accepts a directory path (e.g., `queue/trustlayer/`)
- For each `.md` file: checks all 7 required fields exist
- Checks Status is one of: `draft | approved | sent`
- Checks Channel is one of: `LinkedIn | Email | Both`
- Checks Sequence step is 1, 2, or 3
- Prints PASS/FAIL per file, exits 1 if any file fails

Both scripts should handle empty directories gracefully (no files → exit 0, print "No files found").

**Execution note:** Write these scripts first before running any agents — they define the contract that Researcher and Writer must satisfy. Run them immediately after each agent run to catch drift early.

**Test scenarios:**
- `validate-dossier.sh` on a valid dossier → exit 0, "PASS: jane-doe.md"
- `validate-dossier.sh` on a dossier missing the Hook field → exit 1, "FAIL: jane-doe.md — missing field: Hook"
- `validate-dossier.sh` with ICP Score of "eleven" → exit 1, "FAIL: ICP Score must be numeric 1-10"
- `validate-queue.sh` on a valid queue file → exit 0
- `validate-queue.sh` on a file with Status: `approved` → exit 0 (valid state — Jason sets this manually)
- `validate-queue.sh` on a file with Status: `pending` → exit 1 (not in valid set)
- Empty directory → exit 0, prints "No files found in [path]"
- Scripts are executable (`chmod +x`)

**Verification:** `bash scripts/validate-dossier.sh crm/trustlayer/` and `bash scripts/validate-queue.sh queue/trustlayer/` both exit 0 on valid sample files, exit 1 on intentionally broken test files.

---

### U10. Operator README

**Goal:** Write the README that serves as the operator's startup guide — context switching, weekly workflow, MCP setup, and session startup checklist.

**Requirements:** Self-contained onboarding. Jason (or a future collaborator) can read this and run the system without referring to the spec or plan. Must include the exact steps to switch context and the kill switch locations.

**Dependencies:** U2, U3, U4, U5, U6, U7, U8.

**Files:**
- `claude-agent-protocol/README.md`

**Approach:** Sections:
1. **What this is** (2-3 sentences)
2. **Prerequisites** (Zevari account + Sales Navigator, Gmail MCP auth, Notion MCPs)
3. **Context switching** (edit CLAUDE.md → flip ACTIVE_CONTEXT → for jason mode, also set CURRENT_PROJECT)
4. **Session startup** (checklist: open Claude Code in this directory, confirm context loaded, check kill switches)
5. **Weekly workflow** (4-week rollout table from spec, condensed)
6. **Running each agent** (exact slash commands or session instructions per agent)
7. **Kill switches** (where they are, what they do, how to activate)
8. **Voice DNA** (pointer to `docs/voice-dna-extraction-guide.md`, when to run it)
9. **Validation** (how to run the scripts, what they check)

**Test scenarios:**
- README has all 9 sections
- Prerequisites section lists all 3 required MCP setups
- Kill switches section names both switches and their locations in CLAUDE.md
- Weekly workflow table matches the 4-week plan from the spec

**Verification:** Read README.md cold (pretend no prior context) — can you start the system and run the Researcher without reading anything else? If yes, it's done.

---

## High-Level Technical Design

*This illustrates the intended approach and is directional guidance for review, not implementation specification.*

```
Session start in ~/Code/claude-agent-protocol/
        │
        ▼
Claude reads CLAUDE.md
  ├─ ACTIVE_CONTEXT: trustlayer  →  reads contexts/trustlayer.md
  └─ ACTIVE_CONTEXT: jason       →  reads contexts/jason.md
                                     + contexts/jason/{CURRENT_PROJECT}.md
        │
        ▼
Jason runs an agent command

RESEARCHER                    WRITER                    REPLY TRIAGE
─────────────────────         ─────────────────────     ─────────────────────
Input: LinkedIn URL           Input: crm/{ctx}/*.md     Input: Gmail + LinkedIn
  │                             │                         inbox
  ▼                             ▼                           │
Zevari → Serper → Jina       Reads dossier              Scans threads
  │                           + voice-dna/{ctx}.md       + checks unsubscribe
  ▼                             │                         list
Writes: crm/{ctx}/             ▼                           │
  {name}.md                  Writes: queue/{ctx}/          ▼
                               {name}-{step}.md         Updates: crm/{ctx}/
                                                          {name}.md
                                                        Writes: queue/{ctx}/
                                                          {name}-reply.md
                                                          (if INTERESTED)
        │
        ▼
Jason reviews queue files
  └─ Edits Status: draft → approved
  └─ Sends via Zevari / Gmail (Week 3+)
```

---

## System-Wide Impact

- **New standalone Claude Code project**: Running `claude` in `~/Code/claude-agent-protocol/` produces a different operator experience than running it in `~/Code`. No impact on existing projects.
- **CLAUDE.md precedence**: This CLAUDE.md is independent of `~/.claude/CLAUDE.md` — Alfie identity is not loaded when operating in this project directory. This is intentional — operator mode ≠ Alfie mode.
- **Existing TL-Outbound directory**: `~/Code/TL- Outbound/` exists but is empty (just a README). No conflict. This project supersedes it.
- **MCP availability**: Zevari, Gmail, Notion MCPs must be available in the Claude Code environment where this project runs. If running on a different machine, MCP auth may need to be re-established.

---

## Risk Analysis

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Zevari not yet configured or authenticated | Medium | Blocks Researcher on Week 1 | Set up Zevari auth before Week 1 starts; test with a single profile read |
| Voice DNA extraction too generic | Medium | Writer drafts are mediocre, lose Jason's voice | 20 messages is the minimum; pick from best-performing, not just any sent messages |
| ICP scores poorly calibrated | Medium | Research effort spent on weak prospects | Manually review first 10 dossiers against real pipeline — recalibrate rubric |
| Queue file approved accidentally | Low | Message sent to wrong person or wrong timing | Only Jason can edit Status to `approved`; validate script catches Writer misbehavior |
| UNSUBSCRIBE list not checked before new session | Low–Medium | Contacts known opt-outs | UNSUBSCRIBE guard is in Reply Triage system prompt AND Writer checks it; belt-and-suspenders |
| Context bleed (trustlayer message in jason queue) | Low | Professional/personal cross-contamination | Separate crm/ and queue/ subdirectories per context; validate script checks paths |

---

## Phased Delivery

Following the 4-week rollout from the spec:

| Week | Active | What to test |
|------|--------|--------------|
| 1 | U1–U5, U9–U10 | Researcher on 10 TL prospects. Validate all dossiers. ICP score calibration. |
| 2 | U6–U8 | Extract TL voice DNA. Run Writer on 10 dossiers. Review drafts. Jason context setup. |
| 3 | U7 (Reply Triage) | First 5 LinkedIn sends (approve each manually). First email sends. Triage any replies. |
| 4 | All | Full loop: Research → Write → Approve → Send → Triage. Daily approval cadence. |

The plan covers **Weeks 1 and 2 setup** — Weeks 3-4 activation is system use, not additional build work.

---

## Deferred Implementation Notes

- Exact Zevari MCP tool names and parameter formats — verify against Zevari documentation when setting up in Week 1
- Gmail MCP inbox thread query syntax — test against real inbox before Reply Triage activation
- Sales Navigator-specific Zevari features (lead lists, saved searches) — explore in Week 1 after basic reads work
- Whether `voice-dna/jason.md` should be per-project or shared across all Jason contexts — decide after first extraction run

---

*TDD Iron Law applies during `/work`. Write validation scripts first (U9), then build the content they validate.*
