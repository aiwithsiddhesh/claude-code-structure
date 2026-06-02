---
description: Triage a bug discovered during development or QA. Classifies severity, assigns to the correct agent and sprint, and logs it in SPRINT.md. Use for any defect found during delivery — not just QA bugs.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Bug Triage — $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found."`
```

---

You are the **hiring-manager-orchestrator** working with **manual-functional-sdet**. Triage the reported bug.

---

## Step 1 — Capture Bug Details

Collect from the reporter:
- **What happened**: observed behaviour
- **What should happen**: expected behaviour
- **Steps to reproduce**: exact steps (if known)
- **Where**: feature, component, endpoint, or screen affected
- **Discovered by**: developer self-found / QA / stakeholder / production
- **Frequency**: always / intermittent / once

---

## Step 2 — Classify Severity

| Severity | Definition | Example |
|---|---|---|
| **Critical** | Core functionality broken, no workaround, blocks release | Login fails for all users, data loss, security vulnerability |
| **High** | Feature is unusable but workaround exists, or affects majority of users | Form submits but confirmation never shows, export produces wrong data |
| **Medium** | Degraded experience but feature still works, affects a subset of users | Pagination off by one, tooltip wrong, slow query on edge case |
| **Low** | Cosmetic, copy, or minor UX issue with no functional impact | Button misaligned on mobile, spelling error, wrong colour |

**Additional escalation rules:**
- Any **security vulnerability** → automatically Critical regardless of scope
- Any **data corruption or loss** → automatically Critical
- Any bug that **breaks a committed sprint item's acceptance criteria** → at least High

---

## Step 3 — Assign Owner Agent

Based on where the bug lives:

| Bug Location | Owner Agent |
|---|---|
| UI rendering, component, form, styling | frontend-engineer |
| API response, business logic, auth, DB query | backend-engineer |
| Full-stack data flow (UI → API → DB) | full-stack-engineer |
| CI/CD, deployment, environment, infra | devops-cloud-engineer |
| LLM response, RAG retrieval, prompt | genai-llm-engineer |
| ML model output, data pipeline | data-ml-engineer |
| Cross-cutting or unknown | hiring-manager-orchestrator investigates first |

---

## Step 4 — Assign to Sprint

| Severity | Sprint Assignment |
|---|---|
| **Critical** | Current sprint — pause lower-priority work if needed. Fix before any new features. |
| **High** | Current sprint — add to committed items. Does not pause other work unless capacity is full. |
| **Medium** | Next sprint — add to backlog with High priority tag. |
| **Low** | Backlog — standard priority. Include in next sprint planning if capacity allows. |

**Override rule**: if fixing this bug would require reworking a completed sprint item that manual QA has already signed off, run `/change-request {project-name}` instead — the rework scope needs a formal impact assessment.

---

## Step 5 — Produce Bug Record

```markdown
## Bug BUG-{N}

**Date**: {today}
**Discovered by**: {developer / QA / stakeholder}
**Sprint at discovery**: Sprint {N}

### Description
**Observed**: {what happened}
**Expected**: {what should happen}
**Steps to reproduce**:
1. {step}
2. {step}

**Feature/Component**: {where}
**Frequency**: Always / Intermittent / Once

### Classification
**Severity**: Critical / High / Medium / Low
**Type**: Functional / Security / Performance / UI / Data
**Root cause hypothesis**: {initial guess — agent can revise}

### Assignment
**Owner agent**: {agent}
**Sprint assignment**: Sprint {N} (current) / Sprint {N+1} / Backlog
**Priority within sprint**: Immediate / Normal / Low

### Acceptance Criteria for Fix
- [ ] {specific condition that confirms the bug is resolved}
- [ ] Regression test added to prevent recurrence

### Status
**Current**: Open
```

---

## Step 6 — Update SPRINT.md

Append the bug record to the **Bug Log** section of `output/{project-name}/SPRINT.md`.

If Critical or High assigned to current sprint: also add it to the **Current Sprint** committed items table with `[BUG]` tag.

If Medium or Low going to backlog: add to **Backlog** with `[BUG-{N}]` tag and severity noted.

**Special handling for Critical bugs:**
Output this alert:
```
🚨 CRITICAL BUG — BUG-{N}

This bug is assigned to the current sprint with immediate priority.
{owner-agent} should pause current work and address this first.
hiring-manager-orchestrator should notify all agents working on
dependent features that this may affect their work.
```

Confirm: `✅ Bug BUG-{N} triaged. Severity: {severity}. Assigned to: {agent}. Sprint: {sprint}. SPRINT.md updated.`
