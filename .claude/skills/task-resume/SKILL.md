---
name: task-resume
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
- Check the current Status field:
  - `IN PROGRESS` → proceed normally to Step 2
  - `READY FOR QA` → `✅ This task is already complete. Waiting for manual-functional-sdet sign-off. No work to resume.`
  - `DONE` → `✅ This task is closed. Check SPRINT.md for next assignment.`
  - `ON_HOLD` → `🛑 This task is ON_HOLD — set by the orchestrator. You must NOT work on it. Contact hiring-manager-orchestrator to find out when the hold will be released and what the condition is.`
  - `BLOCKED` → run the BLOCKED resume flow (Step 1B below) before proceeding
  - `REWORK` → run the REWORK resume flow (Step 1C below) before proceeding

### Step 1B — BLOCKED Resume Flow

Read the latest BLOCKED checkpoint entry in TASK.md.

Check whether the blocker condition described in that entry has been resolved:

- **If the blocker is NOT yet resolved** → `⚠️ Task {task-id} is still BLOCKED. Blocker: {description from checkpoint}. Owner: {owner}. Do not start work — the dependency is unresolved. Check back when the blocker owner confirms resolution.`

- **If the blocker IS resolved** → update the Status field in TASK.md from `BLOCKED` to `IN PROGRESS` and append a note to the latest checkpoint:
  ```
  **Blocker resolved**: {today} — {brief description of how it was resolved}
  ```
  Then proceed to Step 2 using the "Context needed to resume after unblocking" from the BLOCKED checkpoint.

### Step 1C — REWORK Resume Flow

Read the latest REWORK checkpoint entry in TASK.md to understand what QA rejected.

Output a REWORK brief before starting:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REWORK: {task-id} — {task description}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

QA REJECTION REASON:
  {copied from REWORK checkpoint}

DEFECTS TO FIX:
  {list of defects and which steps they affect}

STEPS THAT NEED REWORK:
  {list of affected step numbers}

STEPS THAT ARE STILL VALID:
  {list of unaffected step numbers — do not re-implement these}

FILES TO MODIFY:
  {list from REWORK checkpoint}

STARTING WITH:
  {first defect to address}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Update the Status field in TASK.md from `REWORK` to `IN PROGRESS`. Then proceed directly to fixing the first defect.

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
