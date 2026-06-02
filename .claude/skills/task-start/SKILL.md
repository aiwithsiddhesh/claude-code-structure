---
description: Start a sprint task. Breaks it into atomic steps and creates TASK.md before any code is written. Must be run before touching any files for a new task. Use at the beginning of every sprint task session.
disable-model-invocation: true
argument-hint: "[project-name] [task-id]"
---

# Task Start — $ARGUMENTS

## Read Current State

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/SPRINT.md 2>/dev/null || echo "ERROR: SPRINT.md not found"`
```

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/.tasks/$(echo $ARGUMENTS | cut -d' ' -f2).md 2>/dev/null && echo "TASK_EXISTS" || echo "TASK_NOT_FOUND"`
```

---

You are the agent assigned to this task. Work through each step before writing any code.

---

## Step 1 — Validate

- Find the task `{task-id}` in the Current Sprint section of SPRINT.md.
- If not found → stop: `❌ Task {task-id} not found in current sprint. Check SPRINT.md.`
- If TASK.md already exists and has checkpoints → stop: `⚠️ This task already has a TASK.md with progress. Run /task-resume {project} {task-id} instead.`
- If TASK.md already exists but has no checkpoints → confirm: `TASK.md exists with no progress. Starting fresh.`

---

## Step 2 — Understand the Task Fully

From SPRINT.md extract:
- Task description and acceptance criteria
- Dependencies (what must exist before this task starts)
- Which files or endpoints this task will produce

If acceptance criteria are missing or unclear → stop and request clarification from `business-analyst-agent` before proceeding. Do not guess.

---

## Step 3 — Break Into Atomic Steps

Decompose the task into atomic steps. Each step must:
- Produce **one verifiable artifact** (a file, a passing test, a working endpoint)
- Be completable in isolation
- Be checkable without running the whole system

**Atomic step size guide:**
| Too big | Correct size |
|---|---|
| Implement auth | Create users migration |
| Set up database | Write User model with types |
| Build dashboard | Implement GET /api/v1/users endpoint |
| Write tests | Write unit tests for UserService.create() |

**Maximum steps per task: 10.** If you need more than 10, the task is too large — split it and flag to orchestrator.

---

## Step 4 — Identify Context Risk

For each step, assess whether completing it requires reading large existing files:
- If any step needs reading >5 existing files → flag as `HIGH CONTEXT` and note which files
- High context steps should be early in the sequence so they don't get cut off mid-discovery

---

## Step 5 — Create TASK.md

Create `output/{project}/.tasks/{task-id}.md`:

```markdown
# TASK {task-id} — {task description}

**Project**: {project}
**Agent**: {your agent name}
**Sprint**: {sprint number}
**Status**: IN PROGRESS
**Created**: {today}

## Acceptance Criteria
{copied from SPRINT.md — these are what QA will verify}

## Atomic Steps

- [ ] Step 1: {description} → produces: {artifact}
- [ ] Step 2: {description} → produces: {artifact}
- [ ] Step 3: {description} → produces: {artifact}
- [ ] Step N: {description} → produces: {artifact}

## Context Risk
{HIGH CONTEXT steps flagged with which files need reading}
{or: "No high context steps identified"}

## Files This Task Will Write
{list all files that will be created or modified}

## Dependencies
{what must already exist — from other tasks or existing codebase}

## Checkpoints
{written by /task-checkpoint at the end of each incomplete session}

## Completion
{written by /task-checkpoint when all steps are done}
```

---

## Step 6 — Confirm and Begin

Output:
```
✅ TASK.md created for {task-id}.
{N} atomic steps defined.
{N} files will be written.

Starting with Step 1: {description}
```

Begin Step 1 immediately. Do not wait.
