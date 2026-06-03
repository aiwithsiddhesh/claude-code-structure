---
name: sprint-review
description: Close the current sprint. Reviews what was completed vs committed, moves incomplete items back to backlog, updates sprint history in SPRINT.md. Run at the end of every sprint before planning the next one.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Sprint Review — $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found."`
```

---

You are the **hiring-manager-orchestrator**. Work through each step.

---

## Step 1 — Validate Preconditions

- There must be an active sprint (Status = `IN PROGRESS`). If not, stop: `❌ No active sprint found. Run /sprint-plan {project-name} to start one.`

---

## Step 2 — Review Each Committed Item

For each item in the current sprint, determine status from its TASK.md:

| Status | Criteria |
|---|---|
| **DONE** | All steps checked off, Completion entry written, code review APPROVED recorded, `manual-functional-sdet` signed off |
| **READY FOR QA** | All steps checked off, Completion entry written, awaiting `manual-functional-sdet` validation |
| **IN PROGRESS** | TASK.md exists with unchecked steps and at least one checkpoint |
| **BLOCKED** | TASK.md Status field = `BLOCKED` — latest checkpoint has a BLOCKED entry with blocker description and owner |
| **ON_HOLD** | TASK.md Status field = `ON_HOLD` — set by orchestrator, hold condition must be reviewed |
| **REWORK** | TASK.md Status field = `REWORK` — QA rejected, defects documented in latest checkpoint |
| **NOT STARTED** | No TASK.md exists for this item |

**Status conflict check** — after reading each item's TASK.md status, compare it against the status shown in the Current Sprint table in SPRINT.md:

- If both sources agree → proceed with the status as determined.
- If they disagree → flag immediately before proceeding with any other analysis on that item:
  ```
  ⚠️ STATUS CONFLICT on {task-id}: TASK.md says {X}, SPRINT.md says {Y}. Resolve before closing sprint.
  ```
  Do not count a conflicted item as DONE, READY FOR QA, or any other status until the conflict is resolved. Record it in the Sprint Review Report with status `CONFLICT — unresolved`.

For each item marked DONE or READY FOR QA, verify:
- `output/{project}/.tasks/{task-id}.md` exists
- All steps in TASK.md are checked off
- A Completion entry exists (not just a mid-session checkpoint)
- Code review APPROVED is recorded in the Completion entry
- `manual-functional-sdet` sign-off is recorded

If any of these are missing, downgrade the status and note what is missing.

**For every BLOCKED item** — the orchestrator must make an explicit decision. Do not carry it forward silently. Ask:
- Is the blocker resolved? If yes → set back to IN PROGRESS and add to next sprint.
- Is the blocker still unresolved? If yes → record the blocker owner and estimated resolution in the carry-forward note.

**For every ON_HOLD item** — the orchestrator must make an explicit decision. Do not carry it forward silently. Ask:
- Has the hold condition been met? If yes → release the hold, set to IN PROGRESS, carry forward.
- Is the condition still unmet? If yes → confirm ON_HOLD continues into next sprint and state why.

Ask for confirmation on any item where status is unclear before proceeding.

---

## Step 3 — Produce Sprint Review Report

```markdown
## Sprint {N} Review

**Sprint**: {N}
**Dates**: {start} → {end}
**Status**: COMPLETED
**Reviewed**: {today}

### Completion Summary

| ID | Item | Agent | Status | Notes |
|---|---|---|---|---|
| S{N}-1 | {item} | backend-engineer | ✅ DONE | |
| S{N}-2 | {item} | frontend-engineer | ⚠️ IN PROGRESS | 60% done, auth flow incomplete |
| S{N}-3 | {item} | devops-cloud-engineer | 🚫 BLOCKED | Waiting on cloud credentials — owner: PO |
| S{N}-4 | {item} | frontend-engineer | ⏸ ON_HOLD | Hold condition: S{N}-1 must complete first |

### BLOCKED Items — Orchestrator Decision Required

For each BLOCKED item, the orchestrator must decide:

| ID | Blocker | Owner | Decision |
|---|---|---|---|
| S{N}-3 | {blocker description} | {owner} | Carry forward / Drop / Reassign |

### ON_HOLD Items — Orchestrator Decision Required

For each ON_HOLD item, the orchestrator must decide:

| ID | Hold Condition | Met? | Decision |
|---|---|---|---|
| S{N}-4 | {hold condition} | Yes / No | Release / Continue hold |

### Sprint Metrics

- Committed: {N} items
- Completed (DONE): {N} items ({%})
- Ready for QA: {N} items
- In Progress: {N} items
- Blocked: {N} items
- On Hold: {N} items
- Not started: {N} items
- Bugs discovered this sprint: {N} (see Bug Log)

### What Went Well
- [specific observation]

### What Slowed Us Down
- [specific blocker or friction point]

### Carry-Forward Items
Items moved back to backlog for next sprint:
- S{N}-2 (in progress): {remaining work description}
- S{N}-3 (blocked): {blocker status and owner}
- S{N}-4 (on hold): {hold condition and whether it continues}

### Recommended Focus for Next Sprint
[Based on what's in the backlog and what was carried forward, what should sprint {N+1} prioritise?]
```

---

## Step 4 — Update SPRINT.md

Apply these updates to `output/{project-name}/SPRINT.md`:

1. Move current sprint to **Sprint History** section with COMPLETED status and the review summary
2. Move all PARTIAL and NOT STARTED items back to **Backlog** with a note: `[Carry-forward from Sprint {N}]` and what remains
3. Clear the **Current Sprint** section and set: `Sprint {N+1} not started. Run /sprint-plan {project-name} to begin.`
4. Update **Project > Current Sprint** counter to {N+1}

Confirm: `✅ Sprint {N} closed. {complete} of {total} items done. {carry} items returned to backlog. Run /sprint-plan {project-name} to begin Sprint {N+1}.`

---

## Step 5 — Trigger Follow-ups

Based on the review, recommend:
- If completion rate < 70%: `⚠️ Low completion rate. Consider running /change-request {project-name} to reassess scope before Sprint {N+1}.`
- If bugs were discovered: `Run /bug-triage {project-name} for any unresolved bugs before planning next sprint.`
- If this was the final planned sprint: `This was the last planned sprint. Run /release-checklist {project-name} if all features are complete, or reassess timeline.`
