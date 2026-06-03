---
name: task-start
description: Break a sprint task into atomic steps and create TASK.md before any code is written. Run at the very start of every sprint task — before touching any files. Reads the task from SPRINT.md using [project-name] and [task-id] from $ARGUMENTS. Creates output/{project}/.tasks/{task-id}.md with Status IN PROGRESS and begins Step 1 immediately.
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

## Step 2 — Understand the Task Fully and Verify DoR

From SPRINT.md extract:
- Task description and acceptance criteria
- Complexity rating
- Ambiguity rating
- Dependencies (what must exist before this task starts)
- Design Doc column value
- Which files or endpoints this task will produce

Then run the **Definition of Ready (DoR) gate** — verify ALL of the following before proceeding:

| Check | Pass condition |
|---|---|
| Criteria present | At least one acceptance criterion exists |
| Criteria are measurable | No criterion uses vague language ("fast", "user-friendly", "works correctly") — each must name a concrete, verifiable outcome |
| No unresolved open questions | The story contains no `[ ]` open question markers left unanswered |
| Story is correctly sized | ≤ 6 acceptance criteria; maps to one agent end-to-end |
| Design doc cleared (M/L/XL) | If Complexity is Medium, Large, or XL, Design Doc column must show `done` or `waived` — not `pending`, `pending-approval`, `blocked`, or blank |
| Ambiguity resolved | If Ambiguity column is Medium or High, Design Doc column must show `done` or `waived` regardless of Complexity — ambiguity must be resolved even for Small items |
| BA Status ready | BA Status column must show `ready` — not `needs-BA` or `blocked` |

**If the Criteria present, Criteria measurable, No open questions, or Story sized checks fail**, stop immediately:
```
❌ DoR NOT MET — {specific failing check}.

{Explain exactly what is vague, missing, or oversized.}

Return to business-analyst-agent to resolve before starting this task.
Do NOT create TASK.md until DoR passes.
```

**If the Design doc cleared check fails** (Medium/Large/XL, design-doc not done/waived):
```
❌ DoR NOT MET — Design doc required for {complexity} item.
This item shows design-doc: {current value} in SPRINT.md.

Run /design-doc {feature-name} and resolve all open questions and obtain approval before starting this task.
Do NOT create TASK.md until /design-doc is complete and SPRINT.md shows design-doc: done.
```

**If the Ambiguity resolved check fails** (Ambiguity = Medium/High, design-doc not done/waived):
```
❌ DoR NOT MET — Ambiguity unresolved for this item (Ambiguity = {level}).
This item shows design-doc: {current value} in SPRINT.md.

Even Small items require ambiguity resolution when Ambiguity is Medium or High.
Run /design-doc {feature-name} to resolve the ambiguity before starting this task.
Do NOT create TASK.md until /design-doc is complete and SPRINT.md shows design-doc: done.
```

**If the BA Status check fails**:
```
❌ DoR NOT MET — BA Status is {status} for this item.
This item has not been cleared by business-analyst-agent.

Contact business-analyst-agent to complete requirements and set BA Status = ready in SPRINT.md.
Do NOT create TASK.md until BA Status = ready.
```

Do not guess at missing criteria or rephrase vague ones — the BA agent must clarify them.

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

<!-- Valid status values: IN PROGRESS | BLOCKED | REWORK | ON_HOLD | READY FOR QA | DONE
     Transitions: see .claude/rules/team-workflow.md Task State Machine -->

## DoR Verification

**Checked at task start — {today}**

| Check | Result | Notes |
|---|---|---|
| Criteria present | ✅ Pass | |
| Criteria measurable | ✅ Pass | |
| No open questions | ✅ Pass | |
| Story correctly sized | ✅ Pass | |
| Design doc cleared | ✅ Pass / N/A | {value from SPRINT.md, or N/A if Small + Low ambiguity} |
| Ambiguity resolved | ✅ Pass | {Ambiguity level from SPRINT.md} |
| BA Status ready | ✅ Pass | |

**Design doc reference**: {path to design doc, or "N/A — Small item, Low ambiguity" or "N/A — waived by orchestrator on {date}"}

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
