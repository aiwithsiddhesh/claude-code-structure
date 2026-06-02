---
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

## Step 4 — Decision

Based on size and impact, recommend one of three decisions:

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
- Must state the reason clearly

---

## Step 5 — Produce Change Request Record

```markdown
## Change Request CR-{N}

**Date**: {today}
**Project**: {project-name}
**Sprint at time of request**: Sprint {N}
**Requested by**: {source}

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
**Decided by**: hiring-manager-orchestrator [+ product owner if Large/Breaking]
**Reason**: {justification}

**If ABSORB**:
- Agent taking this: {agent}
- Sprint item ID: S{N}-{X} (new) or extends S{N}-{Y}
- Updated sprint end date: {same / extended to {date}}

**If DEFER**:
- Backlog ID: B-{N}
- Priority for next sprint: High / Medium / Low
- Notes for sprint planning: {context}

**If REJECT**:
- Reason: {clear explanation}
- Alternative: {any partial approach or future consideration}
```

---

## Step 6 — Update SPRINT.md

Append the change request record above to the **Change Requests** section of `output/{project-name}/SPRINT.md`.

If ABSORB: also update the **Current Sprint** committed items table with the new or extended item.
If DEFER: add the item to the **Backlog** section with `[CR-{N}]` tag.
If REJECT: no backlog update needed — the record alone is sufficient.

Confirm: `✅ Change Request CR-{N} recorded. Decision: {decision}. SPRINT.md updated.`
