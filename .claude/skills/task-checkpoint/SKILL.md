---
description: Write a checkpoint at the end of a session where the task is not yet complete. Records which steps are done, which files exist, current state, and exact resume instructions. Must be run before ending any session on an incomplete task.
disable-model-invocation: true
argument-hint: "[project-name] [task-id]"
---

# Task Checkpoint — $ARGUMENTS

## Read Current State

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/.tasks/$(echo $ARGUMENTS | cut -d' ' -f2).md 2>/dev/null || echo "ERROR: TASK.md not found. Run /task-start {project} {task-id} first."`
```

---

You are the agent currently working on this task. Write a precise checkpoint before this session ends.

---

## Step 1 — Validate

- TASK.md must exist. If not → `❌ No TASK.md found. Run /task-start {project} {task-id} first.`
- If all steps are already checked off → run completion flow (Step 5) instead of checkpoint.

---

## Step 2 — Assess Current Progress

For each step in TASK.md, determine its status:
- **Done** — artifact exists and is correct
- **Partial** — started but not complete (note exactly what is done and what remains)
- **Not started** — not touched

Be honest. Do not mark a step done if the artifact exists but is broken or incomplete.

---

## Step 3 — List All Files Written This Session

Enumerate every file that was created or modified during this session:
- Full path relative to repo root
- One-line description of what it contains
- Whether it is complete or partial

---

## Step 4 — Write Checkpoint Entry

Append this to the **Checkpoints** section of TASK.md:

```markdown
### Checkpoint {N} — {today}

**Steps completed**: {list step numbers}
**Steps partial**: {list with notes on what remains}
**Steps not started**: {list step numbers}

**Files written this session**:
| File | Status | Notes |
|---|---|---|
| {path} | Complete / Partial | {what it contains} |

**Current system state**:
{Describe what works right now if you ran the code. Example:
"Register endpoint works and returns 201. Login route exists but JWT
signing not implemented. Migration has been run on dev DB."}

**Decisions made this session**:
{Any implementation decisions that the next session must know about.
Example: "Using argon2 not bcrypt — already installed. JWT secret
read from JWT_SECRET env var. Refresh tokens stored in redis not DB."}

**Resume from**:
Step {N}: {exact description of what to do first in next session}

**Context needed to resume**:
{List specific files the next session should read to orient.
Keep this to the minimum — 2-4 files maximum.}

**Open questions**:
{Anything unresolved that will need a decision in the next session.
If none: "None"}

**Blockers**:
{Anything that cannot be resolved without external input.
If none: "None"}
```

---

## Step 5 — Completion Flow (all steps done)

If all steps are checked off, write this instead:

```markdown
### Completion — {today}

**All steps complete**: ✅
**Files written**:
| File | Description |
|---|---|
| {path} | {what it contains} |

**Ready for QA**: Yes
**QA should verify**: {link to acceptance criteria from top of TASK.md}
**Notes for manual-functional-sdet**: {anything specific QA needs to know}
```

Then update the task **Status** field at the top of TASK.md from `IN PROGRESS` to `READY FOR QA`.

Also update SPRINT.md: change the task status to `READY FOR QA` in the Current Sprint table.

---

## Step 6 — Sync SPRINT.md Progress

Update the task row in SPRINT.md Current Sprint table:
- If partial: status = `IN PROGRESS ({N}/{total} steps)`
- If complete: status = `READY FOR QA`

Confirm:
```
✅ Checkpoint {N} written for {task-id}.
Progress: {done}/{total} steps complete.
Resume: /task-resume {project} {task-id}
```
