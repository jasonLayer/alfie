---
name: build
tier: universal
description: The foreman — single entry point for all implementation work. Reads a plan, analyzes task count/dependencies/coupling, and auto-routes to /work (sequential) or /orchestrate (parallel with worktree isolation). Use instead of calling /work or /orchestrate directly. Triggers on "build this", "implement this plan", "start building", or after /plan completes.
argument-hint: "[plan name or path]"
---

# Build: The Foreman

Single entry point for implementation. You hand the foreman a blueprint (the plan), and it decides how to build it.

**Why this exists:** Removes the manual decision between `/work` and `/orchestrate`. The foreman reads the plan, analyzes the tasks, and picks the right execution strategy automatically. You don't tell the foreman *how* to build — you tell it *what* to build.

```
/brainstorm → /plan → /build → /review → /codify → /done
                         │
                    (foreman reads plan)
                    ┌────┴────┐
                  /work    /orchestrate
                            │
                     ┌──────┼──────┐
                   wt/1   wt/2   wt/3
                   (worktree-isolated subagents)
```

## Process

### Step 1: Load the Plan

Find the plan file. Search order:
1. Exact path if provided (e.g., `docs/plans/my-plan.md`)
2. Match by name in `docs/plans/` (e.g., `my-plan` matches `2026-03-17-001-feat-my-plan-plan.md`)
3. If ambiguous, list matches and ask Jason to pick

Read the full plan. Extract:
- All tasks (with their full text, acceptance criteria, files touched)
- The Dependencies section (if present)
- Phase structure (if present)

### Step 1.5: Remote-session preflight

Run the same remote check used elsewhere in Alfie:

```bash
if [ -n "${CLAUDECODE_REMOTE:-}" ] || ! command -v sips >/dev/null 2>&1; then
  echo "remote"
else
  echo "mac"
fi
```

**If `mac`:** skip this step.

**If `remote`:** Some dev-flow steps need a Mac (gstack browser for `/qa` and `/canary`, Preview-flashed Alfie images, Warp panes). Scan the plan and split Mac-only work into a follow-up plan so this build stays focused on what can actually run here.

1. Scan plan tasks for Mac-only markers. Treat as Mac-only if the task:
   - Invokes `/qa` or `/canary`, or describes the work those skills do (end-to-end browser test, post-deploy monitoring loop)
   - Requires manual visual verification ("click through", "screenshot", "visual check", "QA the UI")
   - Requires local-only tools (Warp panes, native apps, hardware, gstack browser)

2. **If ALL tasks are Mac-only:** abort. Tell Jason:
   ```
   This plan is entirely Mac-bound. Either:
     - Finish it on your Mac
     - Kick it off from iPhone via Claude Dispatch to your home Mac
   No build dispatched.
   ```
   Don't proceed to Step 2.

3. **If SOME tasks are Mac-only:** create a sibling follow-up plan at
   `docs/plans/<source-basename>-mac-followup.md` (only if it doesn't already exist; if it does, append new items idempotently). Format:

   ```markdown
   # <Plan name> — Mac follow-up

   Deferred from a remote session on <YYYY-MM-DD>.
   Run these back at the Mac, or kick them off via Claude Dispatch from iPhone.

   Source plan: [<source-basename>.md](<source-basename>.md)

   ## Deferred tasks
   - [ ] **<task ID or short name>** — <one-line context> (originally requires `/qa`)
   - [ ] **<task ID or short name>** — <one-line context> (originally requires `/canary`)
   ```

   Report to Jason in the build announcement:
   ```
   Remote session detected. Mac-only work moved to:
     docs/plans/<source-basename>-mac-followup.md
       - <count> deferred task(s): <one-line summary>
   Continuing build with the remaining <N> task(s).
   ```

   Then continue to Step 2 with **only the non-deferred tasks** considered for routing and dispatch.

### Step 2: Analyze and Route

Score the plan on three dimensions:

**Task count:**
- 1-2 tasks → leans `/work`
- 3+ tasks → leans `/orchestrate`

**Coupling:**
- Parse the Dependencies section for explicit chains (A → B → C)
- Check if tasks share files — if >50% of tasks touch the same files, they're coupled
- If all/most tasks form a single dependency chain → coupled
- If tasks are mostly independent with optional grouping → independent

**Complexity:**
- Are tasks well-defined with clear acceptance criteria? → any strategy works
- Are tasks vague or exploratory? → `/work` (needs human interaction)

**Routing decision:**

| Condition | Route |
|-----------|-------|
| ≤2 tasks | `/work` |
| 3+ tasks, mostly independent, clear acceptance criteria | `/orchestrate` |
| 3+ tasks, but tightly coupled (single chain) | `/work` |
| 3+ tasks, mix of coupled and independent | `/orchestrate` for independent group, `/work` for coupled group (hybrid) |
| Tasks are vague or exploratory | `/work` |

### Step 3: Announce and Confirm

Present the routing decision to Jason:

```
Foreman analysis:
  Tasks: 6 total
  Independent: 4 (tasks 1.1, 1.2, 1.5, 1.6)
  Coupled chain: 2 (tasks 1.3 → 1.4)

  Routing → /orchestrate
    Parallel group: 1.1, 1.2, 1.5, 1.6 (worktree-isolated)
    Sequential group: 1.3 → 1.4 (after dependencies complete)

  Proceed? [Y/override]
```

If Jason says yes → dispatch. If Jason overrides (e.g., "use /work for all of it") → respect the override. The foreman suggests, Jason decides.

### Step 4: Dispatch

**If routing to `/work`:**
- Invoke the CE work skill (`compound-engineering:ce-work`) with the plan
- The foreman steps back — `/work` handles everything from here

**If routing to `/orchestrate`:**
- Invoke `/orchestrate` with the plan
- `/orchestrate` handles worktree isolation, subagent dispatch, two-stage review, compounding
- The foreman monitors for escalations from `/orchestrate`

**If hybrid (mix of coupled and independent):**
1. Dispatch independent tasks to `/orchestrate` (parallel, worktree-isolated)
2. After independent tasks complete, run coupled tasks via `/work` (sequential)
3. Final review covers everything

### Step 5: Handoff

After all work completes (regardless of which strategy was used):

```
Build complete!

Strategy used: /orchestrate (4 parallel) + /work (2 sequential)
Tasks: 6/6 passed
Tests: [test results]

Next: /review for structured review, or /done to close the loop.
```

**If a Mac follow-up plan was created in Step 1.5,** include it in the handoff:

```
Mac follow-up still pending:
  docs/plans/<source-basename>-mac-followup.md (<N> deferred task(s))
  Pick up when you're back at the Mac, or run via Claude Dispatch.
```

## Direct Override

Jason can always bypass the foreman:
- `/work <plan>` — force sequential, interactive mode
- `/orchestrate <plan>` — force parallel subagent mode
- `/build <plan>` — let the foreman decide (default)

The foreman never overrides an explicit choice.

## Edge Cases

- **No plan file found:** Ask Jason to point to the plan or create one with `/plan`
- **Plan has no clear tasks:** Suggest running `/brainstorm` first to clarify scope
- **Plan has 0 tasks completed, 0 remaining:** The plan may already be done — check and confirm
- **Git repo not initialized:** Skip branching, work directly (common for config-only work like ~/.claude/)
- **Mixed testable and non-testable tasks:** TDD applies to code tasks. Config/docs tasks get manual verification.
- **Remote session, all tasks Mac-only:** Step 1.5 aborts the build. No follow-up plan is created (the source plan itself already lists everything Mac-bound).
- **Remote session, follow-up plan already exists:** Append new deferred items to the existing file; don't overwrite. Skip if the item is already listed.
