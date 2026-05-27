---
name: orchestrate
tier: universal
description: Subagent-driven task execution with worktree isolation and two-stage review. Use instead of /work when a plan has 3+ independent tasks. Each subagent works in its own git worktree — code only merges to the feature branch after passing spec + quality review. Dispatches fresh subagent per task, compounds learnings at the end. Triggers on "orchestrate this plan", "run with subagents", or when a plan has many independent tasks.
argument-hint: "[plan name or path]"
---

# Orchestrate: Worktree-Isolated Subagent Execution

Execute a plan by dispatching a fresh subagent per task, each in its own git worktree. Two-stage review after each. Merge only what passes. Compound the learnings.

**Why this exists:** `/work` is great for focused, sequential implementation. `/orchestrate` is for plans with 3+ mostly-independent tasks where you want parallel-safe execution with built-in quality gates — and you want the system to get smarter from the work.

**What makes this safe:** Every subagent works in an isolated copy of the repo (git worktree). Bad code never touches the feature branch. Failed tasks are discarded with zero damage. This is what enables headless mode.

```
feature/my-plan (foreman's branch)
 ├── worktree/task-1 → implement → review passes → merge back ✓
 ├── worktree/task-2 → implement → review fails → fix → re-review → merge ✓
 └── worktree/task-3 → implement → blocked → escalate, worktree discarded
```

## When to Use (vs. /work)

Use `/orchestrate` when:
- The plan has 3+ tasks that are mostly independent
- Tasks touch different files/modules
- You want automated spec + quality review between tasks
- You want to step back and let the system churn
- You want headless-safe execution

Use `/work` when:
- Tasks are tightly coupled (each depends on the last)
- There are only 1-2 tasks
- You want hands-on, interactive implementation

**Note:** You usually don't call `/orchestrate` directly. `/build` (the foreman) reads the plan and routes here automatically when appropriate.

## Process

### Phase 1: Setup

1. Read the plan file (from `docs/plans/` or the provided path)
2. Extract ALL tasks with their full text, acceptance criteria, and context
3. Create a task list tracking all tasks
4. Identify dependencies between tasks — reorder if needed so independent tasks come first
5. Group tasks:
   - **Parallel group:** Independent tasks that can run simultaneously
   - **Sequential group:** Tasks with dependencies (run after their prerequisites complete)
6. Create a feature branch: `git checkout -b <plan-name>`

### Phase 2: Dispatch Loop

For each task:

#### 2a. Dispatch Implementer (Worktree-Isolated)

Spawn a subagent using the Agent tool with `isolation: "worktree"`. This creates an isolated copy of the repo for the subagent to work in.

Construct the prompt from `./implementer-prompt.md`, filling in:
- `{{TASK_TEXT}}` — full task text with acceptance criteria (paste it — don't make the subagent read the plan file)
- `{{CONTEXT}}` — relevant context (which files exist, patterns to follow, results from completed tasks)
- `{{WORKING_DIR}}` — the working directory

```
Agent tool call:
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: [filled implementer-prompt.md]
  description: "Implement task N: [task name]"
```

For independent tasks, dispatch multiple subagents in parallel (multiple Agent tool calls in a single message).

**Handle implementer status:**
- **DONE** → proceed to spec review (in the same worktree)
- **DONE_WITH_CONCERNS** → read concerns. If correctness/scope issue, address before review. If observation, note it and proceed.
- **NEEDS_CONTEXT** → provide missing context, re-dispatch to the same worktree via SendMessage
- **BLOCKED** → assess: provide more context, use a more capable model, break task smaller, or escalate to Jason. If unresolvable, discard the worktree.

#### 2b. Spec Compliance Review

Spawn a reviewer subagent (does NOT need worktree isolation — it reads the worktree's output).

Construct the prompt from `./spec-reviewer-prompt.md`, filling in:
- `{{TASK_TEXT}}` — the original task spec
- `{{IMPLEMENTATION_SUMMARY}}` — the implementer's output (files changed, tests, summary)
- `{{WORKING_DIR}}` — the worktree path (from the implementer's result)

- **COMPLIANT** → proceed to quality review
- **ISSUES** → send issues back to implementer subagent (via SendMessage to the worktree agent), then re-review. Max 2 rounds — after that, escalate to Jason.

#### 2c. Code Quality Review

Spawn a reviewer subagent using `./quality-reviewer-prompt.md`, filling in:
- `{{STYLE_RULES}}` — project style rules from CLAUDE.md (read and paste the relevant section)
- `{{IMPLEMENTATION_SUMMARY}}` — the implementer's output
- `{{WORKING_DIR}}` — the worktree path

- **APPROVED** → merge worktree to feature branch, clean up worktree
- **ISSUES (P1/P2)** → send back to implementer to fix, then re-review. Max 2 rounds.
- **ISSUES (P3 only)** → note them, merge anyway. P3s don't block.

#### 2d. Merge or Discard

**If task passed both reviews:**
1. The worktree has changes on its own branch
2. Merge the worktree branch into the feature branch
3. Clean up the worktree

**If task failed after max review rounds:**
1. Note what failed and why
2. Discard the worktree (no damage to feature branch)
3. Log the failure for Phase 4 compounding

**Handling merge conflicts:**
- Merge worktrees sequentially (even if dispatched in parallel)
- If a conflict occurs: attempt automatic resolution. If complex, escalate to Jason.
- Always run tests on the feature branch after each merge to catch interaction issues.

#### 2e. Progress Update

After each task resolves, one-line update:
```
✓ Task 3/7: Auth middleware — passed spec + quality review, merged
✗ Task 5/7: Email service — blocked (missing SMTP config), worktree discarded
```

### Phase 3: Final Review

After all tasks complete:
1. Switch to the feature branch (all merged worktrees are now here)
2. Run the full test suite — paste output (invoke `/verify` principles)
3. Dispatch a final code quality reviewer across the entire implementation
4. Address any cross-cutting issues found (patterns that only emerge when you see all tasks together)

### Phase 4: Compound

After the work is done:

1. **Identify learnings** from the build:
   - Did any subagent get blocked? Why? Could better context have prevented it?
   - Did spec review catch recurring issues? (e.g., consistently missing error handling)
   - Did quality review flag patterns? (e.g., naming inconsistencies)
   - Were there surprises — things harder or easier than expected?
   - Did any worktree merges conflict? What caused it?

2. **Propose compounding actions** to Jason:
   - Update `CLAUDE.md` with new patterns or rules
   - Write to `docs/solutions/` via `/codify` for reusable insights
   - Update the plan template if the plan format caused confusion
   - Create or update a skill if a workflow pattern emerged
   - Improve the implementer/reviewer prompts based on what they missed

3. **Only apply what Jason approves.** Present the list, let him pick.

### Phase 5: Handoff

```
Orchestration complete!

Branch: <branch-name>
Tasks: 7/7 passed (5 merged, 2 discarded)
Worktrees: 7 created, 5 merged, 2 discarded
Tests: 423 passed, 0 failed

Learnings proposed: 3 (2 approved, applied)

Next: /review for structured review, or /done to close the loop.
```

## Key Rules

- **TDD Iron Law still applies.** Every implementer subagent must write failing tests first. This is non-negotiable — it's in the implementer prompt.
- **Worktree isolation is mandatory.** Every implementer gets `isolation: "worktree"`. No exceptions. This is what makes headless mode safe.
- **Fresh subagent per task.** No context bleed between tasks. You (the orchestrator) curate exactly what each subagent needs.
- **Never skip spec review.** Even if the implementer says "this was straightforward." Trust but verify.
- **Merge sequentially.** Even if tasks run in parallel, merge their worktrees one at a time to catch conflicts early.
- **Max 2 review rounds.** If a task can't pass review after 2 fix cycles, escalate — don't loop forever.
- **Compound every time.** The Phase 4 step is not optional. Even if there are zero learnings, explicitly confirm that.
- **Escalate, don't force.** If a subagent is stuck, don't retry the same thing. Change something (more context, different model, smaller task) or ask Jason.

## Non-Git Repos

If the project is not a git repository (e.g., `~/.claude/` config work):
- Skip worktree isolation — work directly
- Still dispatch fresh subagents per task
- Still run two-stage review
- Still compound learnings
- Note in handoff: "Worktree isolation skipped — not a git repo"

## Prompt Templates

- `./implementer-prompt.md` — dispatch implementer subagent
- `./spec-reviewer-prompt.md` — dispatch spec compliance reviewer
- `./quality-reviewer-prompt.md` — dispatch code quality reviewer
