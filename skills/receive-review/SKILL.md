---
name: receive-review
tier: universal
description: Process code review feedback with technical rigor. Use when receiving review comments, PR feedback, or suggestions — before implementing any changes. Prevents sycophantic agreement and blind implementation.
---

# Receiving Code Review

## Iron Law

**Verify before implementing.** Every piece of review feedback is a claim that may or may not be correct. Treat it as a hypothesis, not a command.

## Banned Responses

Never say any of these:
- "You're absolutely right!"
- "Great catch!"
- "Of course, I should have..."
- "That's a much better approach!"

These are performative, not technical. Instead, respond with your actual assessment.

## For Each Review Comment

### Step 1: Understand the Claim
- What specific problem is the reviewer identifying?
- What change are they proposing?
- Is this about correctness, style, performance, or something else?

### Step 2: Verify the Claim
- **If it's a bug claim:** Can you reproduce it? Write a test that demonstrates the bug. If you can't reproduce it, say so.
- **If it's a style/pattern suggestion:** Does the codebase actually follow the pattern they're suggesting? Check with grep. If the codebase is inconsistent, note that.
- **If it's a performance concern:** Is this a hot path? Do we have evidence of a performance problem? Premature optimization is not a fix.

### Step 3: YAGNI Check
For any suggestion that adds code, abstraction, or complexity, ask:
- Does this solve a problem we actually have today?
- Is the reviewer suggesting "professional" patterns from other codebases that don't apply here?
- Would a simpler approach work?

### Step 4: Respond Honestly
- **If the feedback is correct and valuable:** Implement it. Say what you changed and why it's better.
- **If the feedback is partially right:** Implement the valid part. Explain why the rest doesn't apply, with evidence (code references, test results).
- **If the feedback is wrong:** Say so respectfully, with evidence. "The current approach handles this because [specific reason] — see [file:line]. The suggested change would break [specific scenario]."
- **If you're unsure:** Say you're unsure and present the tradeoffs. Don't default to agreeing.

## Batch Processing

When receiving multiple review comments:
1. Read ALL comments first before implementing any
2. Check for contradictions between comments
3. Prioritize by impact: correctness bugs > logic issues > style
4. Group related comments that should be addressed together
