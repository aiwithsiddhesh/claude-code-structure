# SPRINT.md — {project.name}
<!-- Single source of truth for sprint state. All agents must read this before making decisions. -->

## Project

| Field | Value |
|---|---|
| **Name** | {project.display_name or project.name} |
| **Start Date** | {today} |
| **Deadline** | {timeline.deadline} |
| **Sprint Length** | 1 week |
| **Total Sprints** | {calculated: deadline / sprint length, rounded up} |
| **Current Sprint** | 0 — not started |

---

## Current Sprint

> No sprint active. Run `/sprint-plan {project.name}` to commit Sprint 1.

---

## Backlog

All features from project intake. Items are pulled into sprints via `/sprint-plan`.
Bugs and change requests are appended here automatically by `/bug-triage` and `/change-request`.

| ID | Item | Source | Priority | Notes |
|---|---|---|---|---|
| B-1 | {feature 1 from intake} | intake | - | |
| B-2 | {feature 2 from intake} | intake | - | |
| B-N | {feature N from intake} | intake | - | |

---

## Change Requests

| ID | Date | Description | Size | Decision | Sprint |
|---|---|---|---|---|---|
| — | | | | | |

---

## Bug Log

| ID | Date | Description | Severity | Agent | Sprint | Status |
|---|---|---|---|---|---|---|
| — | | | | | | |

---

## Sprint History

| Sprint | Dates | Committed | Completed | Carry-forward | Notes |
|---|---|---|---|---|---|
| — | | | | | |
