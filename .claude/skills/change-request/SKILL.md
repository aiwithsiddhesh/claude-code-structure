---
name: change-request
description: Assess and decide a requirement change mid-sprint. Use whenever a stakeholder requests a new feature, a requirement changes, or scope needs to shift after a sprint has started. Appends decision to SPRINT.md.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Change Request — $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found."`
```

---

You are the **hiring-manager-orchestrator**. Guide a structured change assessment.

---

## Step 1 — Capture the Change

Ask the user to describe the change if not already stated. You need:

- **What changed**: the original requirement and what it is now
- **Why**: business reason or stakeholder driving it
- **Source**: who is requesting (stakeholder, product owner, discovered during dev)
- **Urgency**: does this block current sprint work or can it wait?

## Step 1B — Duplicate CR Detection

Before proceeding, scan the **Change Requests** table in SPRINT.md for any existing CR that:
- Requests the same or substantially overlapping change
- Affects the same features or items
- Was submitted by the same source recently (same sprint)

If a likely duplicate is found:
```
⚠️ POSSIBLE DUPLICATE CHANGE REQUEST — CR-{N} may already cover this request.

Existing CR: CR-{N} — {description}
Status: {decision or pending}
Similarity: {what overlaps}

Options:
1. If this is the same change: reference CR-{N} rather than creating a new record.
2. If this is a different change: confirm "create new CR" to proceed.
3. If the prior CR was rejected and this is a re-submission: confirm and note it as a re-submission of CR-{N}.
```

Wait for confirmation before creating a new CR record.

---

## Step 2 — Classify Change Size

Assess the change against what is currently committed in the sprint:

| Size | Definition |
|---|---|
| **Trivial** | Wording change, UI copy, colour, label — no logic change. No impact on sprint. |
| **Small** | < 2 hours of work, no new agents needed, doesn't touch other committed items |
| **Medium** | 2–8 hours, may affect 1–2 committed items, same agents already assigned |
| **Large** | > 8 hours, or requires reworking already completed items, or needs a new agent |
| **Breaking** | Invalidates core design decisions already implemented — requires sprint reset |

---

## Step 3 — Impact Assessment

For every committed sprint item that this change touches, state:
- Does it need to be **redone** (work already done is now wrong)?
- Does it need to be **extended** (more work added to an in-progress item)?
- Does it **block** another committed item?
- Does it require a **new agent** not currently assigned?

Also assess: does this change conflict with any rule in `.claude/rules/` or any constraint in `project-intake.json`?

---

## Step 4 — Decision Authority and SLA

Before making a decision, verify the authority requirements for this change size:

| Change Size | Required Decision-Makers | If Unavailable |
|---|---|---|
| Trivial / Small | Orchestrator alone — decide this session | N/A |
| Medium | Orchestrator + product owner must agree | If product owner unavailable this sprint → auto-defer |
| Large / Breaking | Orchestrator + product owner required | Sprint may need to pause; replan via `/sprint-plan` after decision |
| Reject (any size) | Orchestrator alone | Record reason in SPRINT.md with enough detail requestor understands why |

**Response time SLA** — classify the impact level of this change:

| Impact Level | Definition | SLA |
|---|---|---|
| Critical | Would block current sprint delivery | Decide before any agent continues on affected tasks — this session |
| High | Affects 2+ committed sprint items | Decide within current sprint — cannot carry into next sprint undecided |
| Medium | Affects 1 committed item | Decide by end of current sprint |
| Low | Does not affect any committed item | May defer to next `/sprint-plan` — must be recorded in backlog |

**In-progress work rules during review:**

| Change Size | Action on In-Progress Work |
|---|---|
| Trivial / Small | Work continues uninterrupted |
| Medium | Move affected tasks to `ON_HOLD` status immediately — orchestrator sets the hold |
| Large / Breaking | Move ALL dependent tasks to `ON_HOLD` immediately |

If any tasks need to be set to `ON_HOLD`, list them explicitly and instruct the agent(s) to update their TASK.md Status field before this session ends.

---

Based on size, authority, and impact, recommend one of three decisions:

**ABSORB** — take it into the current sprint
- Only valid for Trivial or Small changes
- Only if sprint capacity allows (no agent is already at full load)
- Only if it does not require reworking completed items
- Document which agent takes the additional work

**DEFER** — move to backlog for next sprint
- Valid for any size change that doesn't block current sprint delivery
- The change is logged, prioritised, and will be included in next `/sprint-plan`
- Current sprint continues unchanged

**REJECT** — not doing this work
- Valid when change contradicts `project-intake.json` out_of_scope
- Valid when change would cause timeline to miss the project deadline
- Valid when product owner decides the value doesn't justify the cost
- Must state the reason clearly — recorded in SPRINT.md for requestor visibility

---

## Step 5 — Produce Change Request Record

```markdown
## Change Request CR-{N}

**Date**: {today}
**Project**: {project-name}
**Sprint at time of request**: Sprint {N}
**Requested by**: {source}
**Re-submission of**: CR-{prior N} / New request

### What Changed
**Original**: {original requirement}
**Proposed**: {new requirement}
**Reason**: {business justification}

### Impact Assessment
| Affected Item | Impact Type | Effort |
|---|---|---|
| {item} | Rework / Extend / Block | {hours} |

**Change size**: Trivial / Small / Medium / Large / Breaking
**New agents required**: Yes ({agent}) / No
**Conflicts with constraints**: Yes ({detail}) / No

### Decision

**Decision**: ABSORB / DEFER / REJECT
**Impact level**: Critical / High / Medium / Low
**SLA**: {required decision timeline per impact level}

**Options considered**:
1. {Option A — e.g., Absorb into current sprint}
2. {Option B — e.g., Defer to next sprint}
3. {Option C — e.g., Reject — out of scope}

**Chosen option**: {which option}
**Rationale**: {specific reasoning — why this option over the others; must not be generic}
**Decided by**: {orchestrator alone / orchestrator + product owner name}
**Decision date**: {today}

**Tasks set to ON_HOLD pending this decision**:
{List task IDs and agents, or "None — Trivial/Small change, work continues"}

**If ABSORB**:
- Agent taking this: {agent}
- Sprint item ID: S{N}-{X} (new) or extends S{N}-{Y}
- Updated sprint end date: {same / extended to {date}}
- Tasks released from ON_HOLD: {list or "N/A"}

**If DEFER**:
- Backlog ID: B-{N}
- Priority for next sprint: High / Medium / Low
- Notes for sprint planning: {context}
- Tasks released from ON_HOLD: {list — deferred change unblocks them}

**If REJECT**:
- Reason: {clear explanation specific enough for requestor to understand}
- Alternative: {any partial approach or future consideration}
- Tasks released from ON_HOLD: {list — rejection unblocks them}
```

---

## Step 6 — Update SPRINT.md

Append the change request record above to the **Change Requests** section of `output/{project-name}/SPRINT.md`.

Also update the **Human Decisions** table in SPRINT.md with a one-line entry:
```
| CR-{N} | Change request: {brief description} | {ABSORB/DEFER/REJECT} | {options listed} | {chosen option} | {rationale summary} | {decider} | {today} |
```

If ABSORB: also update the **Current Sprint** committed items table with the new or extended item.
If DEFER: add the item to the **Backlog** section with `[CR-{N}]` tag.
If REJECT: no backlog update needed — the Human Decisions entry is sufficient.

Confirm: `✅ Change Request CR-{N} recorded. Decision: {decision}. SPRINT.md updated.`
