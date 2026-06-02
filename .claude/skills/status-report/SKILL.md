---
description: Generate a project status report. Use when providing a progress update on an active project.
disable-model-invocation: true
argument-hint: "[project name]"
---

# Project Status Report: $ARGUMENTS

Generate a structured status report covering all active agents and work streams.

## Status Report

```markdown
# Project Status Report: [Project Name]
Date: [today]
Reporting Period: [e.g., Week 2 of 6]
Overall Status: 🟢 On Track | 🟡 At Risk | 🔴 Blocked

---

## Summary (2–3 sentences)
[High-level state of the project: what's going well, what's at risk, key decision needed.]

## Progress by Agent

| Agent | Status | Completed This Period | In Progress | Upcoming |
|---|---|---|---|---|
| [agent] | 🟢/🟡/🔴 | [work done] | [current work] | [next up] |

## Milestone Tracker

| Milestone | Target Date | Status | Notes |
|---|---|---|---|
| [milestone] | [date] | On Track / At Risk / Delayed | [comment] |

## Blockers

| # | Description | Impact | Owner | Target Resolution |
|---|---|---|---|---|
| 1 | [blocker] | High/Med/Low | [agent] | [date] |

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
```
