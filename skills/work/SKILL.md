---
name: work
tier: universal
description: Execute an implementation plan sequentially with TDD enforcement. Use when the user says "work on the plan", "start working", "implement this", or provides a plan and wants sequential interactive execution. Forces TDD Iron Law regardless of CE engine behavior.
argument-hint: "[plan name or path]"
---

Start working on the plan: $ARGUMENTS

## Core Engine

**Invoke the Compound Engineering work skill now:**

```
skill: compound-engineering:ce-work
args: $ARGUMENTS
```

Follow the CE work process completely — plan reading, environment setup, todo creation, task execution with system-wide test checks, incremental commits, quality checks, and shipping. The CE engine handles all of this.

## Alfie Overrides & Additions

### TDD Iron Law (MANDATORY)

This overrides any CE behavior that would skip tests:

- **No production code without a failing test first.**
- If code is written before the test, delete it and start over.
- Every task during work starts with a failing test.
- This is non-negotiable regardless of what the CE engine suggests.

### Plan File Updates

As work progresses, update the plan file by checking off completed items (`[ ]` to `[x]`). The CE engine already does this, but ensure it happens for every task.

### Definition of Done

Before creating PR, verify these Alfie-specific items in addition to CE's quality checklist:
- [ ] TDD was followed for every task (failing test first)
- [ ] Plan file checkboxes are all checked off
- [ ] Code is readable for team onboarding — clear naming, no clever tricks
