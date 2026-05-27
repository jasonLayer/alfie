---
name: verify
tier: universal
description: Verification before claiming work is complete. Use before saying "done", "fixed", "passing", "working", committing, or creating PRs. Evidence before assertions — always.
---

# Verification Before Completion

## Iron Law

**Never claim success without evidence in the same message.** If you say "tests pass" you must have run them and shown the output. If you say "it works" you must have executed it. No exceptions.

## Before Claiming Done

Run through this checklist:

### 1. Tests
- [ ] Run the specific tests for the code you changed — paste the output
- [ ] Run the broader test suite — paste the output (or confirm count: "854 passed, 0 failed")
- [ ] If you wrote new tests, confirm they fail without your change (red-green)

### 2. Functionality
- [ ] If it's a backend change: hit the endpoint with a real request (curl, httpie, or test)
- [ ] If it's a frontend change: load the page and verify visually (or note that you can't)
- [ ] If it's a bug fix: reproduce the original bug scenario and confirm it's resolved

### 3. Regressions
- [ ] Check that related features still work (not just the thing you changed)
- [ ] If you changed a shared utility/service, grep for other callers and verify they're unaffected

### 4. Cleanup
- [ ] No debug logging left in code
- [ ] No commented-out code added
- [ ] No unrelated changes in the diff

## Delegating to Subagents

If you dispatched work to a subagent, you MUST verify their output yourself. Run the tests. Read the diff. Don't trust "all tests pass" from a subagent without seeing evidence.

## The Rule of Proof

When reporting completion to the user:
- **Do:** "All 856 tests pass (output above). The endpoint returns the correct response (curl output above)."
- **Don't:** "Everything should be working now." / "Tests should pass." / "This should fix it."

"Should" is not evidence. Run it.
