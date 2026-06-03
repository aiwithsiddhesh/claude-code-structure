---
name: sprint-plan
description: Plan and commit the next sprint. Reads SPRINT.md backlog, checks BA readiness and ambiguity for each item, assigns items to agents, sets sprint dates, and updates SPRINT.md. Run at the start of every sprint cycle after BA readiness is confirmed.
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

1. **No active sprint**: Status must not be `IN PROGRESS`. If there is an active sprint, stop: `❌ Sprint {N} is still active. Run /sprint-review to close it before planning the next sprint.`

2. **No duplicate sprint in progress**: If the Current Sprint section already shows a sprint number and dates matching today, stop: `❌ Sprint {N} was already committed this session. Do not run /sprint-plan again without first running /sprint-review.`

3. **Backlog not empty**: If backlog is empty, output: `✅ Backlog is empty. All work is complete. Consider running /retrospective {project-name}.`

---

## Step 2 — Determine Sprint Number and Dates

- New sprint number = last completed sprint + 1 (or 1 if no sprints yet)
- Sprint start = today
- Sprint end = today + sprint length (from SPRINT.md project config)
- If sprint end exceeds project deadline → flag as `⚠️ TIMELINE RISK: this sprint end date ({date}) is past the project deadline ({deadline}). Consider reducing scope.`

---

## Step 3 — BA Readiness Check

**This check runs before any item may be committed.** It is not advisory.

For each backlog item being considered for this sprint, check its **BA Status** column in SPRINT.md:

| BA Status | Meaning | Sprint eligibility |
|---|---|---|
| `ready` | Acceptance criteria written, DoR passed | Eligible for sprint commit |
| `needs-BA` | No acceptance criteria yet | NOT eligible — cannot be committed |
| `blocked` | BA work is blocked on a dependency | NOT eligible — cannot be committed |

If any candidate item has BA Status `needs-BA` or `blocked`:

```
⚠️ SPRINT READINESS BLOCK — {N} item(s) not ready for sprint commitment:

{For each blocked item:}
  - {item-id} ({item name}): BA Status = {status}
    Missing: {acceptance criteria / open questions / BA decision}
    Action: business-analyst-agent must complete requirements and update SPRINT.md BA Status to `ready` before this item can be committed.

Items with BA Status = needs-BA or blocked will NOT be committed to this sprint.
Only items with BA Status = ready may proceed to the ambiguity check below.
```

Continue sprint planning with only the `ready` items. Do not silently include `needs-BA` items.

---

## Step 4 — Assess Backlog and Commit Scope

Review each `ready` backlog item. For each one, assess:

- **Complexity**: Small (< 1 day) / Medium (1–3 days) / Large (3–5 days) / XL (> 5 days)
- **Ambiguity**: Low / Medium / High (assess from the user story, acceptance criteria, and open questions)
- **Dependencies**: does it require another item to be done first?
- **Agent**: which agent handles it (use `.claude/rules/agent-coordination.md`)?

Record Complexity and Ambiguity in the sprint item row.

Recommended sprint capacity per agent per week: 3–4 Medium items, or 1–2 Large items. Do not overcommit.

Select items to commit this sprint based on:
1. Priority (Critical bugs first, then High, then features by dependency order)
2. Dependencies (don't commit item B if item A it depends on isn't also committed)
3. Capacity (don't assign more than an agent can realistically handle)

Items NOT committed stay in the backlog for the next sprint — do not remove them.

---

## Step 5 — Design and Ambiguity Gate

Apply to every committed item. This gate is based on Complexity AND Ambiguity — both dimensions matter.

**Design doc requirement by Complexity:**

| Complexity | Design doc required? |
|---|---|
| Small | Not required unless Ambiguity is Medium/High |
| Medium | Required — mark `design-doc: pending` unless already done |
| Large | Required — mark `design-doc: pending` unless already done |
| XL | Required — mark `design-doc: pending` unless already done |

**Ambiguity override — applies regardless of Complexity:**

If **Ambiguity = Medium or High** on any item, regardless of Complexity (including Small):
- A `/design-doc` run is required to resolve the ambiguity before development starts
- Mark `design-doc: pending` and emit:
  ```
  ⚠️ AMBIGUITY REVIEW REQUIRED: {item-id} ({complexity}, Ambiguity={level}) — run /design-doc before any development begins on this item. Small items are not exempt when ambiguity is unresolved.
  ```

**For each committed item, check whether `design-doc: done` or `design-doc: waived` is already recorded:**
- **If not recorded and design doc required** → add `design-doc: pending` and emit the DESIGN REQUIRED or AMBIGUITY REVIEW REQUIRED warning.
- **If `design-doc: done`** → no action needed.
- **If `design-doc: waived`** → orchestrator has explicitly waived. Record the waiver in SPRINT.md Human Decisions table with: reason, approver, date. Waiver is not valid without this record.
- **If `design-doc: blocked`** → item may be committed but CANNOT start development. `/design-doc` must be unblocked first.

A sprint with pending design docs is valid to commit — the design gate blocks `/task-start`, not sprint planning. But the sprint plan output must clearly list all pending design docs.

---

## Step 6 — Produce Sprint Plan

Output this sprint plan:

```markdown
## Sprint {N} Plan

**Sprint**: {N}
**Dates**: {start} → {end}
**Status**: IN PROGRESS

### Committed Items

| ID | Item | Complexity | Ambiguity | Agent | Dependencies | Design Doc | Acceptance Criteria | Status | Notes |
|---|---|---|---|---|---|---|---|---|---|
| S{N}-1 | {item} | Medium | Low | backend-engineer | none | pending | {criteria} | IN PROGRESS | |
| S{N}-2 | {item} | Small | High | frontend-engineer | S{N}-1 | pending | {criteria} | IN PROGRESS | Ambiguity review required |

### Agent Assignments This Sprint

| Agent | Items | Notes |
|---|---|---|
| [agent] | S{N}-1, S{N}-2 | [any coordination notes] |

### Pending Design Docs (must complete before /task-start on these items)
{List all items with design-doc: pending. If none: "None — all design gates cleared."}

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

## Step 7 — Update SPRINT.md

Rewrite the **Current Sprint** section of `output/{project-name}/SPRINT.md` with the plan above.
Move committed items from **Backlog** to the **Current Sprint** section with all columns populated: ID, Item, Complexity, Ambiguity, Agent, Dependencies, Design Doc, Acceptance Criteria, Status, Notes.

Do not modify Sprint History, Change Requests, or Bug Log sections.
Do not remove unclaimed backlog items.
Update the BA Status of committed items to preserve their `ready` status in the sprint table.

If any waivers were granted, update the Human Decisions table in SPRINT.md.

Confirm: `✅ Sprint {N} committed. {count} items assigned across {agent count} agents. {N} design docs pending. Run /status-report {project-name} to track progress.`

**Remind all assigned agents:**
```
For each task assigned to you this sprint:
1. Run /task-start {project} {task-id} before writing any code
2. Run /task-checkpoint {project} {task-id} before ending any incomplete session
3. Run /task-resume {project} {task-id} at the start of any new session on existing work

Tasks live at output/{project}/.tasks/{task-id}.md
Design docs (if required) must be completed before /task-start — check Design Doc column.
```
