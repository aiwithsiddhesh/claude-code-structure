---
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

For each item in the current sprint, determine status:

**DONE** — implementation complete, code reviewed, manual QA validated
**PARTIAL** — started but not complete (note: what % done, what remains)
**NOT STARTED** — not touched this sprint
**BLOCKED** — work started but stopped due to dependency, bug, or external blocker

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
| S{N}-2 | {item} | frontend-engineer | ⚠️ PARTIAL | 60% done, auth flow incomplete |
| S{N}-3 | {item} | devops-cloud-engineer | ❌ NOT STARTED | blocked on cloud credentials |

### Sprint Metrics

- Committed: {N} items
- Completed: {N} items ({%})
- Partial: {N} items
- Not started: {N} items
- Bugs discovered this sprint: {N} (see Bug Log)

### What Went Well
- [specific observation]

### What Slowed Us Down
- [specific blocker or friction point]

### Carry-Forward Items
Items moved back to backlog for next sprint:
- S{N}-2 (partial): {remaining work description}
- S{N}-3 (not started): {reason}

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
