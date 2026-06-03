---
name: status-report
description: Generate a structured project status report for an active project. Use when a progress update is needed — weekly, before a stakeholder check-in, or when the orchestrator needs a snapshot of all work streams. Reads SPRINT.md, all active TASK.md files, and all open BUG-N.md files. Produces a report covering overall status, per-agent progress, milestone tracker, blockers, risks, and decisions needed.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Project Status Report: $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found. Check project name."`
```

```
!`ls output/$ARGUMENTS/.tasks/*.md 2>/dev/null | head -20 || echo "NO_TASKS_YET"`
```

```
!`for f in output/$ARGUMENTS/.tasks/*.md; do [ -f "$f" ] && echo "=== $f ===" && cat "$f"; done 2>/dev/null || echo "NO_TASK_FILES"`
```

```
!`for f in output/$ARGUMENTS/.bugs/BUG-*.md; do [ -f "$f" ] && echo "=== $f ===" && cat "$f"; done 2>/dev/null || echo "NO_BUG_FILES"`
```

---

You are the **hiring-manager-orchestrator**. Generate a status report from all three sources above: SPRINT.md (sprint-level state), TASK.md files (true task execution state), and BUG-N.md files (true bug lifecycle state).

**Do not report task status from SPRINT.md alone.** SPRINT.md is authoritative for sprint membership and priority; TASK.md is authoritative for task execution progress. If they disagree, report the conflict.

**`/status-report` is an observability command, not a blocking gate.** It continues and completes the report even when STATUS CONFLICTS are found — conflicts are surfaced prominently in the report so the orchestrator can act on them. The blocking behaviour on STATUS CONFLICT applies to action-taking skills (`/task-start`, `/task-resume`, `/release-checklist`, `/sprint-review`) — not to reporting.

---

## Step 1 — Detect Status Conflicts

Before generating the report, compare task statuses across sources:

For each task in the Current Sprint:
1. Read its Status from SPRINT.md
2. Read its Status from TASK.md (if it exists)

If they disagree:
```
⚠️ STATUS CONFLICT detected on {task-id}:
  SPRINT.md says: {status}
  TASK.md says: {status}
  
This conflict must be resolved. Recording in report under State Conflicts.
True status per TASK.md: {status from TASK.md}
```

For each bug in the Bug Log:
1. Read its Status from SPRINT.md Bug Log
2. Read its Status from BUG-N.md

If they disagree, report similarly.

---

## Step 2 — Generate Status Report

```markdown
# Project Status Report: [Project Name]
Date: [today]
Reporting Period: [e.g., Week 2 of 6]
Overall Status: 🟢 On Track | 🟡 At Risk | 🔴 Blocked

---

## Summary (2–3 sentences)
[High-level state of the project: what's going well, what's at risk, key decision needed.]

## Progress by Agent

| Agent | Status | Completed This Period | In Progress | Steps Done/Total | Upcoming |
|---|---|---|---|---|---|
| [agent] | 🟢/🟡/🔴 | [work done] | [current task + step count from TASK.md] | [N/M] | [next up] |

*Step counts are read from TASK.md — not inferred from SPRINT.md.*

## Task Detail (from TASK.md)

| Task ID | Description | Status (TASK.md) | Status (SPRINT.md) | Steps | Last Checkpoint | Conflict? |
|---|---|---|---|---|---|---|
| [id] | [desc] | [TASK.md status] | [SPRINT.md status] | [done/total] | [date] | ✅ / ⚠️ CONFLICT |

## Open Bugs (from BUG-N.md)

| Bug ID | Title | Severity | Status (BUG.md) | Status (SPRINT.md) | Assigned | Sprint | Age | Conflict? |
|---|---|---|---|---|---|---|---|---|
| [id] | [title] | Critical/High/Medium/Low | [BUG.md status] | [SPRINT.md status] | [agent] | [N] | [days open] | ✅ / ⚠️ CONFLICT |

*Status values read from BUG-N.md: OPEN | IN FIX | READY FOR VERIFICATION | REJECTED | VERIFIED | CLOSED*

## State Conflicts

| Source | ID | SPRINT.md Status | True Status (TASK/BUG) | Action Required |
|---|---|---|---|---|
| [TASK/BUG] | [id] | [sprint status] | [true status] | [what must be resolved] |

*If no conflicts: "None — all statuses consistent."*

## Milestone Tracker

| Milestone | Target Date | Status | Notes |
|---|---|---|---|
| [milestone] | [date] | On Track / At Risk / Delayed | [comment] |

## Blockers

| # | Description | Impact | Owner | Task/Bug ID | Duration | Target Resolution |
|---|---|---|---|---|---|---|
| 1 | [blocker] | High/Med/Low | [agent] | [id] | [days blocked] | [date] |

*Blockers sourced from TASK.md BLOCKED checkpoints and BUG-N.md status.*

## Escalation Flags

Items requiring immediate orchestrator attention:
- Any Critical/High bug OPEN or IN FIX for more than 1 sprint
- Any task BLOCKED for more than 1 sprint
- Any task IN PROGRESS across 2+ sessions without a completion checkpoint
- Sprint completion rate below 70%

[List flagged items, or "None — no escalation triggers active."]

## Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| [risk] | High/Med/Low | High/Med/Low | [action] |

## Decisions Needed

| Decision | Owner | Deadline | Context |
|---|---|---|---|
| [decision] | [person] | [date] | [brief context] |

## Next Period Plan
- [Key actions planned for the next reporting period]
- [...]

---

**Prepared by**: hiring-manager-orchestrator
**Sources read**: SPRINT.md + {N} TASK.md files + {N} BUG-N.md files
```
