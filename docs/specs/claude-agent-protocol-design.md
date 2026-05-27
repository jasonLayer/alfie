# Claude Agent Protocol — Personal GTM System Design

**Status**: Draft  
**Author**: Jason Reichl  
**Date**: 2026-05-11  
**Framework source**: Peslar Claude Agent Protocol

---

## What We're Building

A file-system-native, multi-agent GTM automation system that runs inside Claude Code. Three parallel agents coordinate via flat files (no event bus, no server) to research prospects, draft outreach, and triage replies across LinkedIn and email.

The system supports two contexts:
- **TrustLayer** — Jason as AE/operator selling TrustLayer, targeting COI compliance buyers in construction/real estate
- **Jason** — Personal GTM for Jason's ventures (8thday, POAM, MajorGTM, etc.), context set per session

The same codebase runs both. A single `ACTIVE_CONTEXT` variable in CLAUDE.md determines which context is live.

---

## Why

The Peslar article demonstrates that Claude doesn't need to be a chatbot to run outreach workflows — it can operate as a coordinated agent system where each agent has a narrow job, reads from flat files, and writes structured outputs. This is:

1. **Faster to ship** than building a custom app (pure Claude Code + filesystem)
2. **More controllable** than SaaS sequencers (you own the voice DNA, ICP, and logic)
3. **Testable in isolation** — run Researcher alone before wiring Writer
4. **Portable** — the filesystem/agent pattern can be imported into MajorGTM later

---

## Dual-Context Architecture

### CLAUDE.md Structure

```
ACTIVE_CONTEXT: trustlayer   # flip to: jason
CURRENT_PROJECT: —           # (jason mode only) e.g. 8thday, poam, majorgtm
```

Claude reads `ACTIVE_CONTEXT` at the top of every session and loads the corresponding context file from `contexts/`. All agent prompts, ICP, voice DNA, CRM path, and Notion routing are pulled from the active context.

### Context Files

**`contexts/trustlayer.md`**
- ICP: Mid-market construction/real estate/property operations leaders managing 50+ subcontractor COIs
- Pain: Manual certificate verification, compliance gaps, liability when cert lapses
- Titles to target: VP Operations, Director of Risk, Procurement Manager, General Contractor owners
- Company size: 100–2,000 employees
- Voice: Precise, peer-to-peer, no vendor speak, COI/compliance vocabulary
- CRM path: `crm/trustlayer/`
- Queue path: `queue/trustlayer/`
- Email: jason@trustlayer.io
- LinkedIn: Jason Reichl (TrustLayer)
- Notion: TrustLayer workspace (mcp__notion-trustlayer__)

**`contexts/jason.md`**
- CURRENT_PROJECT: set at runtime (e.g., `majorgtm`, `poam`, `8thday`, `revconsulting`)
- ICP: Loaded per project from sub-files (`contexts/jason/{project}.md`)
- Email: jason@8thday.io (or project-appropriate)
- LinkedIn: Jason Reichl (personal) + Sales Navigator
- Notion: Personal workspace (mcp__notion-personal__)

**Jason project ICP files (`contexts/jason/`):**
- `majorgtm.md` — GTM founders/operators building outbound systems; feel the pain of manual outreach and tool sprawl
- `poam.md` — Art community contacts: curators, collectors, venue directors, fellow artists in the Bay Area creative scene
- `8thday.md` — Executives/founders seeking 1:1 coaching; burnt out high-achievers who want clarity on work and life
- `revconsulting.md` — RevOps leaders and CROs at B2B SaaS companies needing AI workflow design and GTM architecture help

---

## Agent Definitions

### Agent 1: Researcher
**Role**: Read-only prospect intelligence. No outreach, no writes to queue.  
**Input**: LinkedIn URL or `{name} + {company}`  
**Output**: `/crm/{context}/{firstname-lastname}.md` dossier  

**Dossier format**:
```
# {Name} — {Title} @ {Company}
## ICP Score: {1-10}
## Hook: {one sentence — specific, not generic}
## LinkedIn: {url}
## Email: {if found}
## Company snapshot: {3 sentences — size, focus, recent news}
## Pain signals: {what points to COI/compliance pain or equivalent}
## Objection forecast: {likely pushback}
## Recommended approach: {LinkedIn | Email | Both}
## Research date: {YYYY-MM-DD}
```

**Tools**: Zevari LinkedIn MCP (read), Serper search, Jina reader, Playwright (fallback)

### Agent 2: Writer
**Role**: Read dossiers, write outreach drafts. Never sends. Writes to queue only.  
**Input**: `/crm/{context}/{name}.md`  
**Output**: `/queue/{context}/{name}-{sequence-step}.md`  

**Queue file format**:
```
# Outreach Draft — {Name}
## Channel: LinkedIn | Email
## Sequence step: {1|2|3}
## Status: draft | approved | sent
## Subject/Note: {subject line or connection note}
## Body: {message}
## Personalization hook used: {from dossier}
## Voice DNA check: {passed | flagged — reason}
```

**Voice DNA constraint**: Every draft is checked against `/voice-dna/{context}.md` before writing. Writer flags and rewrites any phrase that violates voice rules. Never proceeds with a flagged draft without annotation.

**Tools**: Read dossiers, write queue files, read voice DNA

### Agent 3: Reply Triage
**Role**: Monitor inbox, classify replies, update CRM, surface hot leads.  
**Input**: Gmail/LinkedIn inbox  
**Output**: Updated CRM dossier + `/queue/{context}/{name}-reply.md` for suggested response  

**Classification**:
- `INTERESTED` — book meeting, fast-path to calendar
- `NOT NOW` — update dossier with timing signal, set follow-up date
- `WRONG PERSON` — note referral if given, mark dossier
- `UNSUBSCRIBE` — flag immediately, never contact again
- `NO REPLY` — handled by sequence step logic

**Tools**: Gmail MCP, Zevari LinkedIn MCP (inbox read), write CRM, write queue

---

## File System Layout

```
~/Code/claude-agent-protocol/
├── CLAUDE.md                        ← ACTIVE_CONTEXT + CURRENT_PROJECT
├── contexts/
│   ├── trustlayer.md
│   └── jason/
│       ├── 8thday.md
│       ├── poam.md
│       └── majorgtm.md
├── voice-dna/
│   ├── trustlayer.md                ← extracted from Jason's 20 best TL messages
│   └── jason.md                     ← extracted from Jason's 20 best personal messages
├── crm/
│   ├── trustlayer/                  ← {firstname-lastname}.md per prospect
│   └── jason/
├── queue/
│   ├── trustlayer/                  ← drafts staged for review
│   └── jason/
├── agents/
│   ├── researcher.md                ← researcher system prompt + audit template
│   ├── writer.md                    ← writer system prompt + queue format
│   └── reply-triage.md              ← triage system prompt + classification rules
└── docs/
    ├── specs/                       ← this document
    └── plans/
```

---

## MCP / Tool Stack

| Tool | Purpose | Cost |
|------|---------|------|
| **Zevari** | LinkedIn reads + sends (rate-limited, session-safe) | $87/mo |
| **Gmail MCP** | Email sends, inbox monitoring | Existing |
| **mcp__notion-trustlayer__** | TrustLayer Notion reads/writes | Existing |
| **mcp__notion-personal__** | Personal Notion reads/writes | Existing |
| **Serper** | Web search for prospect research | Existing/low cost |
| **Jina** | Read web pages (LinkedIn, company sites) | Existing |
| **Google Calendar MCP** | Meeting booking for INTERESTED replies | Existing |
| **Playwright** | Fallback browser for unstructured pages | Existing |

---

## Voice DNA Extraction Process

Before Writer agent is activated (Week 2), Jason runs a one-time extraction:

1. Collect 20 best-performing LinkedIn/email messages for each context (TL, Jason)
2. Run extraction prompt:
   ```
   Analyze these 20 messages. Extract:
   - Tone adjectives (3-5)
   - Sentence length pattern (avg words, range)
   - Opening styles that appear (hook types)
   - Phrases that appear 3+ times
   - Words/phrases to NEVER use (corporate, generic, cringe)
   - Punctuation and formatting style
   - How I end messages
   Save as voice-dna/{context}.md
   ```
3. Jason reviews and edits before Writer uses it

---

## 4-Week Rollout

**Week 1 — Researcher Only**
- Set up project structure and CLAUDE.md
- Activate TrustLayer context
- Run Researcher manually on 10 TL prospects
- Validate dossier quality and ICP score calibration
- No outreach

**Week 2 — Writer + Voice DNA**
- Extract TL voice DNA from Jason's best messages
- Activate Writer
- Review 10 queue drafts — edit DNA as needed
- No sends yet
- Begin Jason context setup in parallel

**Week 3 — Reply Triage + First Sends**
- Activate Reply Triage
- First 5 LinkedIn sends (manual approve each)
- First email sends (manual approve each)
- Triage any replies
- Calibrate sequence timing

**Week 4 — Autonomous Loop**
- Full loop: Research → Write → Approve → Send → Triage
- Jason approves queue items daily (not individual steps)
- Weekly: review CRM health, ICP score distribution, reply rates

---

## Notion Routing

| Session context | Notion workspace | Operation |
|----------------|-----------------|-----------|
| ACTIVE_CONTEXT: trustlayer | mcp__notion-trustlayer__ | Look up ICP, save specs/plans/learnings |
| ACTIVE_CONTEXT: jason | mcp__notion-personal__ | Look up project context, save to personal workspace |

---

## TrustLayer ICP (Loaded in trustlayer.md)

**Primary buyer**: Operations Director / VP of Operations at general contractors and subcontractor-heavy construction or property management companies  
**Secondary buyer**: Risk Manager, Compliance Manager, Procurement Director  
**Company profile**: 100–2,000 employees, mid-market, manages 50+ active subcontractors or vendors  
**Pain**: COI collection is manual (email/spreadsheet), certs expire without warning, GC gets exposed when a subcontractor's policy lapses, insurance broker involvement creates delays  
**Trigger events**: New compliance requirement from a GC client, insurance renewal season, a near-miss liability incident, growth to new markets requiring new vendor compliance standards  
**Voice match**: Direct, compliance-vocabulary-comfortable, respects time, peer-not-vendor framing  

---

## What This Is Not

- Not a SaaS product (no backend, no hosted service, no billing)
- Not a replacement for CRM systems (it's a research + drafting layer, exports to HubSpot when needed)
- Not autonomous without Jason's daily approve step (safety by design)
- Not the full MajorGTM scope (that's a future integration, not this build)

---

## Decisions

1. **Zevari + Sales Navigator**: Jason has Sales Navigator — Zevari will use it for richer prospect data and higher send limits.
2. **Reply Triage trigger**: On-demand for MVP. Jason runs `/reply-triage` as a Claude command when ready to check inbox. No polling, no server, no webhooks. Graduate to cron after Week 4 if the system proves out.
3. **Jason context projects**: All four active in Week 2 — MajorGTM, POAM art contacts, 8thday coaching, RevOps/AI consulting.

## Open Questions

1. **CRM sync**: When do dossiers get pushed to HubSpot vs stay flat files forever?
2. **MajorGTM integration path**: Flat file bus → shared module? Or just shared conventions when the time comes?

---

## Success Metrics

- Week 1: 10 dossiers created, ICP scores feel calibrated (Jason validates manually)
- Week 2: Voice DNA extracted, Writer produces drafts Jason would send without edits
- Week 3: First replies tracked, triage classifications accurate
- Week 4: Jason spending <15 min/day on outreach review

---

*Next step: `/plan claude-agent-protocol` to turn this into an implementation plan*
