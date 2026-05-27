---
name: invest
tier: personal
config: [trustlayer_investment_data]
description: Run a TrustLayer investment analysis session. Reads live dashboard metrics, fetches external fundraising and comp data, generates multi-angle analysis, supports chat corrections, persists context between sessions, and exports to Notion on demand. Use when Jason says "/invest" or "run investment analysis".
argument-hint: "[optional: 'export' to push to Notion immediately]"
---

# /invest — Investment Analysis

Run a full investment analysis session for TrustLayer. This skill auto-reads dashboard metrics, fetches external data, generates investment angles, enters a chat correction loop, and persists everything for the next session.

---

## Setup

**Working directory:** `~/Code/TL-3vc_tool`

**Key files:**
- Input: `output/dashboard_data.json` — live metrics
- Context store: `output/investment-context.json` — persistent corrections across sessions
- Analysis output: `output/investment-analysis.md` — what the dashboard reads
- Session history: `output/investment-history/YYYY-MM-DD.json` — per-session snapshots

**Notion export target:** Investment Analysis section — page ID `362fe491-b55f-8152-bcbc-cc06c0cc771e` (in TrustLayer workspace, use `mcp__notion-trustlayer__` tools).

---

## Phase 1: Data Read

Read `output/dashboard_data.json`. Extract:
- **KPIs block**: EoP ARR, YoY ARR growth, NDR, renewal rate, gross churn, expansion rate, Win Rate (blended)
- **Channel summary**: opportunity counts and win rates for Direct / Referral / Partner
- **Investments block** (if present): avg deal size, pricing tiers, account count, ARR trajectory
- **`generated` timestamp**: if older than 90 days from today, warn — "Data is N days old — analysis may not reflect current state." Continue regardless.

If `output/dashboard_data.json` does not exist: stop and say "Run the dashboard server first — `python3 server.py` in ~/Code/TL-3vc_tool."

---

## Phase 2: Context Load

Check for `output/investment-context.json`.

**If it exists:** Read it. Surface a brief preamble: "Carrying forward from prior sessions: [bullet list of positioning overrides, pricing corrections, proof points]." Then merge prior corrections into your working context for this session.

**If it does not exist:** Initialize with empty schema — do not create the file yet, just hold the schema in memory:
```
{
  "positioning": {},
  "pricing_model": {},
  "product_roadmap": "",
  "proof_points": [],
  "corrections_history": [],
  "last_updated": null,
  "metrics_snapshot": {}
}
```

**If the file exists but is malformed JSON:** Warn — "Context file is corrupted — starting fresh this session. Prior corrections will not carry forward." Initialize with empty schema.

---

## Phase 3: External Fetch

Run two web searches using available search tools:

1. **Fundraising history:** "TrustLayer insurance startup funding rounds investors Series A B"
2. **SaaS comp multiples:** "SaaS Series B valuation multiples [current ARR range] ARR growth 2024 2025"

If searches fail or return no meaningful signal: proceed with dashboard data only. Note at the top of the analysis: "External market data unavailable — analysis based on dashboard metrics only."

---

## Phase 4: Analysis Generation

Generate **4–5 investment angles** based on what the data actually warrants — not a fixed template. Let the metrics drive what's interesting. Each angle must include:
- **Thesis:** what the investment story is for this angle
- **Math:** specific numbers from the dashboard supporting the thesis
- **Key risks:** 1–3 honest risks or open questions
- **Verdict:** bull/bear/neutral signal for this angle

**Known context to incorporate (from prior sessions or this session):**
- TrustLayer is a **RIMIS** (Risk Information Management Intelligence System) — NOT a carrier API endpoint
- **PLG motion**: vendor invitation network drives organic expansion
- **Pricing model**: Starter (free COI tracking) → Compliance app at $5-7/vendor/year Basic, $8-10 Pro, $15-20 Complete
- **Automation App** is live and servicing accounts
- **App roadmap**: Compliance → Automation → Contract Profiler → Insurance Inbox → Web Forms → Document Extraction

Add this label at the top of the analysis output: **"Draft — correct any assumptions that are off."**

---

## Phase 5: Chat Correction Loop

Enter an interactive chat loop. Accept natural language corrections from Jason.

For each correction:
1. **Acknowledge** — "Got it — updating [angle / assumption]."
2. **Update** — fold the correction into the relevant angle(s) and regenerate or revise that section
3. **Persist** — append to the in-memory `corrections_history` array: `{ "timestamp": "<ISO datetime>", "correction": "<text>", "applied_to": ["<angle titles>"] }`

Loop continues until Jason types one of: `done`, `export`, or `exit`.

- `done` → exit loop, proceed to Phase 6 (write outputs, no Notion export)
- `export` → exit loop, proceed to Phase 6 + Phase 8 (write outputs AND export to Notion)
- `exit` → exit loop, ask "Write outputs before exiting?" If yes, proceed to Phase 6. If no, stop.

---

## Phase 6: Session Versioning

Create the history directory if it doesn't exist: `output/investment-history/`

Write a session snapshot to `output/investment-history/YYYY-MM-DD.json` (use today's date). If a file for today already exists, append `-v2`, `-v3`, etc.

Snapshot schema:
```json
{
  "date": "YYYY-MM-DD",
  "metrics_snapshot": {
    "arr": "<EoP ARR from dashboard>",
    "arr_growth": "<YoY growth>",
    "ndr": "<NDR>",
    "renewal_rate": "<renewal rate>",
    "win_rate_blended": "<blended win rate>"
  },
  "angles": ["<angle title 1>", "<angle title 2>", ...],
  "corrections_applied": ["<correction text>", ...],
  "analysis_text": "<full analysis markdown>"
}
```

---

## Phase 7: Output Write

Write `output/investment-analysis.md` with:

```markdown
# Investment Analysis
**Date:** YYYY-MM-DD
**Metrics baseline:** ARR [value], NDR [value], [corrections applied: N]

---

[full analysis body with all angles]
```

Overwrite any existing file.

Then write/merge `output/investment-context.json` — merge corrections from this session into the existing context file (or create it if absent):
- New keys in `positioning`, `pricing_model`, `product_roadmap`, `proof_points` overwrite old values
- `corrections_history` always appends — never overwrites
- `last_updated` = ISO datetime now
- `metrics_snapshot` = current session metrics

---

## Phase 8: Notion Export (only on `export`)

Push the analysis to Notion using `mcp__notion-trustlayer__API-post-page`:
- **Parent**: `{"page_id": "362fe491-b55f-8152-bcbc-cc06c0cc771e"}`
- **Title**: `Investment Analysis — YYYY-MM-DD`
- **Children**: Convert the analysis markdown to Notion blocks (headings, bullets, paragraphs)

Display the returned page URL: "Exported → [URL]"

---

## Completion

After writing all outputs, report:
```
Analysis complete.
  Angles: [N]
  Corrections applied: [N]
  Output: output/investment-analysis.md
  Context saved: output/investment-context.json
  Session: output/investment-history/YYYY-MM-DD.json
  [Export: <Notion URL>  ← only if exported]

View in dashboard: Investment tab at http://localhost:5173/
```
