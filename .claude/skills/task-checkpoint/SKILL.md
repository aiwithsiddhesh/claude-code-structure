---
name: task-checkpoint
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
- Check current Status field. If `ON_HOLD` → `❌ This task is ON_HOLD. Only the orchestrator can release it. Do not write a checkpoint — contact hiring-manager-orchestrator.`

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

**If the task is being BLOCKED this session**, use the BLOCKED checkpoint format:

Update the **Status** field in TASK.md to `BLOCKED` and append this to the **Checkpoints** section:

```markdown
### Checkpoint {N} — {today} [BLOCKED]

**Steps completed**: {list step numbers}
**Steps partial**: {list with notes on what remains}
**Steps not started**: {list step numbers}

**Blocker description**: {What exactly cannot proceed — be precise. Example: "Cannot implement payment webhook without Stripe test keys. Dev environment has no access to credentials store."}

**What is needed to unblock**: {Specific thing needed — a decision, credentials, a completed task, external access. Example: "Stripe test API key added to .env.example and shared via password manager."}

**Blocker owner**: {Who is responsible for resolving this — agent name, product owner, or external party.}

**Estimated resolution**: {Best estimate of when the blocker will be resolved. Example: "Sprint end — PO to confirm Stripe account access."}

**Files written this session**:
| File | Status | Notes |
|---|---|---|
| {path} | Complete / Partial | {what it contains} |

**Context needed to resume after unblocking**:
{List specific files the next session should read. 2-4 files maximum.}
```

---

**If the task is in REWORK this session** (QA rejected), use the REWORK checkpoint format:

Update the **Status** field in TASK.md to `REWORK` (set by `manual-functional-sdet`, not the dev agent) and append this:

```markdown
### Checkpoint {N} — {today} [REWORK]

**QA rejection reason**: {Exact reason from manual-functional-sdet — what failed, what the expected behaviour was.}

**Defects to fix**:
- Defect 1: {description} → affects Step {N}
- Defect 2: {description} → affects Step {N}

**Steps affected by rework**: {list step numbers that need to be redone or modified}

**Steps unaffected**: {list step numbers that remain valid and do not need rework}

**Files to modify**:
{List files that need to change to address the defects.}

**Resume from**:
{Exact description of what to do first — which defect to tackle first and why.}
```

---

**Standard checkpoint** (task is IN PROGRESS, not blocked or rework):

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
{Anything that cannot be resolved without external input. If this field is non-empty, run the BLOCKED checkpoint flow instead of this one.
If none: "None"}
```

---

## Step 5 — Completion Flow (all steps done)

**Duplicate completion guard:** Before writing a Completion entry, check whether one already exists in the TASK.md Completion section. If a Completion entry is already present (contains "### Completion"), stop:
```
⚠️ Completion entry already exists for {task-id}. This task was already marked complete on {date from existing entry}.
If the task was rejected by QA and is in REWORK, do not write a new Completion entry — fix the defects first, then re-run /task-checkpoint.
If this is an error, check TASK.md manually.
```

If all steps are checked off and no Completion entry exists, **run `/code-review {project} {task-id}` first** before writing the Completion entry.

- If code review result is `CHANGES REQUESTED` → do NOT write a Completion entry. Fix the identified issues, then re-run `/code-review`.
- If code review result is `APPROVED` → write the Completion entry below.

```markdown
### Completion — {today}

**All steps complete**: ✅
**Code review**: APPROVED
**Code review notes**: {brief summary — what was checked, any notable findings that were addressed}

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
