---
name: sprint-plan
description: Plan and commit the next sprint. Reads SPRINT.md backlog, assigns items to agents, sets sprint dates, and updates SPRINT.md. Run at the start of every sprint cycle.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Sprint Planning — $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found. Run /start-project first."`
```

---

You are the **hiring-manager-orchestrator**. Work through each step using the SPRINT.md state above.

---

## Step 1 — Validate Preconditions

Before planning a new sprint, verify:
- There is no currently active sprint (Status is not `IN PROGRESS`). If there is an active sprint, stop and output: `❌ Sprint {N} is still active. Run /sprint-review to close it before planning the next sprint.`
- The backlog has at least one item. If backlog is empty, output: `✅ Backlog is empty. All work is complete. Consider running /retrospective {project-name}.`

---

## Step 2 — Determine Sprint Number and Dates

- New sprint number = last completed sprint + 1 (or 1 if no sprints yet)
- Sprint start = today
- Sprint end = today + sprint length (from SPRINT.md project config)
- If sprint end exceeds project deadline → flag as `⚠️ TIMELINE RISK: this sprint end date ({date}) is past the project deadline ({deadline}). Consider reducing scope.`

---

## Step 3 — Assess Backlog and Commit Scope

Review each backlog item. For each one, assess:
- **Complexity**: Small (< 1 day) / Medium (1–3 days) / Large (3–5 days) / XL (> 5 days)
- **Dependencies**: does it require another item to be done first?
- **Agent**: which agent handles it (use `.claude/rules/agent-coordination.md`)?

Recommended sprint capacity per agent per week: 3–4 Medium items, or 1–2 Large items. Do not overcommit.

Select items to commit this sprint based on:
1. Priority (Critical bugs first, then High, then features by dependency order)
2. Dependencies (don't commit item B if item A it depends on isn't also committed)
3. Capacity (don't assign more than an agent can realistically handle)

Items NOT committed stay in the backlog for the next sprint — do not remove them.

**Design gate — apply to every committed item:**

| Complexity | Design requirement |
|---|---|
| Small | No design doc required |
| Medium | `/design-doc` required — mark `design-doc: pending` unless already done |
| Large | `/design-doc` required — mark `design-doc: pending` unless already done |
| XL | `/design-doc` required — mark `design-doc: pending` unless already done |

For each Medium/Large/XL item being committed, check whether `design-doc: done` or `design-doc: waived` is already recorded on the item in SPRINT.md:
- **If not recorded** → add `design-doc: pending` to the item row and emit: `⚠️ DESIGN REQUIRED: {item-id} ({complexity}) — run /design-doc before any development begins on this item.`
- **If `design-doc: done`** → no action needed.
- **If `design-doc: waived`** → the orchestrator has explicitly waived the requirement. Record the waiver reason in SPRINT.md if not already present.

A sprint with Medium/Large/XL items all showing `design-doc: pending` is still valid to commit — the design gate blocks `/task-start`, not sprint planning. But the sprint plan output must clearly list all pending design docs so the orchestrator knows what must be done before development begins.

---

## Step 4 — Produce Sprint Plan

Output this sprint plan:

```markdown
## Sprint {N} Plan

**Sprint**: {N}
**Dates**: {start} → {end}
**Status**: IN PROGRESS

### Committed Items

| ID | Item | Complexity | Agent | Dependencies | Design Doc | Acceptance Criteria |
|---|---|---|---|---|---|---|
| S{N}-1 | {item} | Medium | backend-engineer | none | pending | {criteria} |
| S{N}-2 | {item} | Small | frontend-engineer | S{N}-1 | n/a | {criteria} |

### Agent Assignments This Sprint

| Agent | Items | Notes |
|---|---|---|
| [agent] | S{N}-1, S{N}-2 | [any coordination notes] |

### Sprint Goal
[One sentence: what does a successful sprint look like?]

### Risks
- [Any risk that could prevent sprint completion]

### Definition of Done for This Sprint
- [ ] All committed items implemented and code-reviewed
- [ ] No open Critical or High bugs
- [ ] Manual QA validated committed items
- [ ] Handoffs completed between dependent agents
```

---

## Step 5 — Update SPRINT.md

Rewrite the **Current Sprint** section of `output/{project-name}/SPRINT.md` with the plan above.
Move committed items from **Backlog** to the **Current Sprint** section.
Do not modify Sprint History or Change Requests sections.
Do not remove unclaimed backlog items.

Confirm: `✅ Sprint {N} committed. {count} items assigned across {agent count} agents. Run /status-report {project-name} to track progress.`

**Remind all assigned agents:**
```
For each task assigned to you this sprint:
1. Run /task-start {project} {task-id} before writing any code
2. Run /task-checkpoint {project} {task-id} before ending any incomplete session
3. Run /task-resume {project} {task-id} at the start of any new session on existing work

Tasks live at output/{project}/.tasks/{task-id}.md
```
