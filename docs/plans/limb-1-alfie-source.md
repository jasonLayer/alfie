# Plan: LIMB 1 — Alfie as DO source

**Goal:** When I defer or skip something in an Alfie session, Alfie creates a JR_DO_DB row with `Source=Alfie` and useful context, so it lives in the pile and shows up in tomorrow's Captured list.

**Pre-req:** Spine shipped (do-capture skill exists, JR_DO_DB writable).

**Definition of Done:**
- Alfie reliably proposes capture at natural deferral moments without nagging.
- When I say "yes, capture it" (or it's auto-captured per a configured rule), a JR_DO_DB row appears with `Source=Alfie` and a body containing the context (what we were doing, why deferred, file/branch if relevant).
- Alfie can mark previously-captured items done when I confirm they're handled.

---

## Tasks

### 1. Define deferral triggers
Document the natural stopping points where capture should be proposed:
- I say variations of "later", "not now", "skip", "park that", "save for later".
- A plan task is explicitly skipped during /work or /orchestrate.
- A branch is abandoned with TODOs left behind.
- A bug is identified that's out of scope for the current session.

Document the anti-triggers (don't propose capture):
- Information-only Q&A.
- Mid-flow exploration where I'm comparing options.
- When I'm clearly in a stream of consciousness.

### 2. Capture proposal voice
- Short, one-line proposal: "Capture '{summary}' to DO?"
- If I say yes, fire `create_do`.
- If I'm silent or say no, don't ask again about the same thing this turn.
- Never block on the proposal — keep moving.

### 3. Context block
When creating the row, the body should contain:
- One-sentence summary of what we were doing.
- Why it's being deferred (in Alfie's voice — Adlerian framing OK).
- File path / branch name / PR link if available.
- Timestamp + session marker.

Use the `context` parameter on `create_do`.

### 4. Completion side
When I tell Alfie "I handled X" or "X is done" and X matches an open DO row, call `mark_done(X)`. Confirm with a one-liner.

### 5. Behavioral guardrails
- Don't propose more than 2 captures per turn.
- Don't propose capture during high-flow moments (active TDD loop, mid-debug).
- Remember within a session what's already been captured; don't re-propose.

### 6. Update memory file
Add a "LIMB 1: Alfie source — operational rules" subsection to `memory/jr-do-system.md` covering triggers, anti-triggers, and voice.

### 7. Test
- Mid-session, say "let's skip that one for now."
- Confirm Alfie proposes capture, I say yes, a Source=Alfie row appears.
- Tomorrow, run "build today's log" and confirm the row appears in Captured Today.
- Tell Alfie "I handled the thing about X" — confirm row gets marked done.

---

## Risk / Open Questions

- **Annoyance threshold:** capture proposals are net-negative if they interrupt flow. The default should be conservative — better to miss captures than to nag. Iterate after a week of real use.
- **Context length:** keep the body under ~3 short paragraphs. If more is needed, it's probably a Plan, not a DO row.
