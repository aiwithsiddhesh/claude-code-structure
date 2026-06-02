---
description: Resume an incomplete task in a new session. Reads the last checkpoint from TASK.md and orients the agent to continue from exactly where the previous session ended. Always run this first when continuing work on an existing task.
disable-model-invocation: true
argument-hint: "[project-name] [task-id]"
---

# Task Resume — $ARGUMENTS

## Read Task State

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/.tasks/$(echo $ARGUMENTS | cut -d' ' -f2).md 2>/dev/null || echo "ERROR: TASK.md not found at output/{project}/.tasks/{task-id}.md"`
```

---

You are the agent resuming this task. Orient completely from TASK.md before writing a single line of code.

---

## Step 1 — Validate

- TASK.md must exist. If not → `❌ No TASK.md found. Was /task-start run? Check output/{project}/.tasks/`
- Task status must be `IN PROGRESS`. If status is:
  - `READY FOR QA` → `✅ This task is already complete. Waiting for manual-functional-sdet sign-off.`
  - `DONE` → `✅ This task is closed. Check SPRINT.md for next assignment.`
  - `BLOCKED` → `⚠️ This task is blocked. Read the blocker note in the latest checkpoint and resolve before continuing.`

---

## Step 2 — Build Resume Brief

Read the latest Checkpoint entry in TASK.md and produce a structured resume brief:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RESUMING: {task-id} — {task description}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PROGRESS: {done}/{total} steps complete

DONE:
  ✅ Step 1: {description}
  ✅ Step 2: {description}

REMAINING:
  ⬜ Step 3: {description} → produces: {artifact}
  ⬜ Step 4: {description} → produces: {artifact}

FILES ALREADY WRITTEN:
  {path} — {what it contains}
  {path} — {what it contains}

CURRENT STATE:
  {copied from last checkpoint's "Current system state"}

DECISIONS ALREADY MADE:
  {copied from last checkpoint's "Decisions made" — do not re-decide these}

STARTING NOW:
  Step {N}: {exact description from resume-from field}

CONTEXT TO READ FIRST:
  {files listed in "Context needed to resume" — read these, then start}

OPEN QUESTIONS TO RESOLVE:
  {any open questions from last checkpoint}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 3 — Read Context Files

Read only the files listed in **Context needed to resume** from the last checkpoint. Do not read the full codebase. The checkpoint was written specifically to minimize what you need to re-read.

If a listed file does not exist → flag immediately:
`⚠️ Expected file {path} does not exist. Previous session may have ended before writing it. Review Step {N} in TASK.md and re-implement.`

---

## Step 4 — Continue From Resume Point

Begin the step listed in **STARTING NOW** immediately. Do not:
- Re-implement steps already marked done
- Re-read files not in the context list
- Re-make decisions already recorded in TASK.md
- Ask for clarification on anything already decided in a previous checkpoint

If you discover that a completed step's artifact is broken or missing:
- Note it explicitly: `⚠️ Step {N} was marked done but {artifact} is missing/broken. Re-implementing.`
- Uncheck that step in TASK.md
- Fix it before proceeding

---

## Step 5 — Checkpoint Before This Session Ends

When this session ends with the task still incomplete:
Run `/task-checkpoint {project} {task-id}` — do not skip this.

Confirm start:
```
▶ Resuming {task-id} from Step {N}.
  {N} steps done, {N} remaining.
  Reading: {context files}
  Then: {first action}
```
