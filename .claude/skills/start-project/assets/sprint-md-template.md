# SPRINT.md — {project.name}
<!-- Single source of truth for sprint state. All agents must read this before making decisions. -->
<!-- Precedence: SPRINT.md is authoritative for sprint membership, priority, assignment, and release scope.
     TASK.md is authoritative for task execution details and checkpoint history.
     BUG-N.md is authoritative for bug lifecycle details.
     If status conflicts, the skill must stop and report STATUS CONFLICT before proceeding. -->

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

## Readiness Issues

Record BA readiness failures, unresolved open questions, and DoR blocks that prevent backlog items from being committed to a sprint. Cleared when issue is resolved.

| ID | Item | Issue | Owner | Resolved |
|---|---|---|---|---|
| — | | | | |

---

## Design Decisions

Record design approvals, waivers, and BLOCKED states here. Updated by `/design-doc` when a design is approved or blocked.

| Item ID | Feature | Design Doc Path | Status | Approver | Date | Notes |
|---|---|---|---|---|---|---|
| — | | | pending | | | |

---

## Human Decisions

Every human decision recorded here with full context. Updated by `/change-request`, `/design-doc`, and `/sprint-plan` when a decision requiring human judgment is made.

| ID | Context | Decision | Options Considered | Chosen Option | Rationale | Decider | Date |
|---|---|---|---|---|---|---|---|
| — | | | | | | | |

---

## State Conflicts

Skills must stop and record here when SPRINT.md and TASK.md/BUG-N.md status disagree. Cleared when resolved.

| Task/Bug ID | SPRINT.md Status | TASK/BUG Status | Reported By | Resolved |
|---|---|---|---|---|
| — | | | | |

---

## Current Sprint

> No sprint active. Run `/sprint-plan {project.name}` to commit Sprint 1.

---

## Backlog

All features from project intake. Items are pulled into sprints via `/sprint-plan`.
Bugs and change requests are appended here automatically by `/bug-triage` and `/change-request`.

| ID | Item | Source | Priority | Complexity | Ambiguity | BA Status | Acceptance Criteria | Notes |
|---|---|---|---|---|---|---|---|---|
| B-1 | {feature 1 from intake} | intake | - | - | - | needs-BA | | |
| B-2 | {feature 2 from intake} | intake | - | - | - | needs-BA | | |
| B-N | {feature N from intake} | intake | - | - | - | needs-BA | | |

**BA Status values**: `needs-BA` | `ready` | `blocked`
**Ambiguity values**: `Low` | `Medium` | `High`

---

## Change Requests

| ID | Date | Description | Size | Decision | Options Considered | Chosen Option | Rationale | Decider | Sprint |
|---|---|---|---|---|---|---|---|---|---|
| — | | | | | | | | | |

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
