---
name: bug-triage
description: Triage a bug discovered during development or QA. Classifies severity, assigns to the correct agent and sprint, creates BUG-{N}.md as the living bug document, and logs a reference in SPRINT.md. Use for any defect found during delivery — not just QA bugs.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Bug Triage — $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: SPRINT.md not found"`
```

```
!`ls output/$ARGUMENTS/.bugs/ 2>/dev/null | grep "^BUG-" | sort -t'-' -k2 -n | tail -1 || echo "NO_BUGS_YET"`
```

---

You are the **hiring-manager-orchestrator** working with **manual-functional-sdet**. Triage the reported bug.

---

## Step 1 — Capture Bug Details

Collect from the reporter:
- **What happened** — observed behaviour
- **What should happen** — expected behaviour
- **Steps to reproduce** — exact steps (if known). If unknown, ask manual-functional-sdet to reproduce before proceeding.
- **Where** — feature, component, endpoint, or screen affected
- **Discovered by** — developer self-found / QA / stakeholder
- **Frequency** — always / intermittent / once

Do not proceed until reproduction steps are known. A bug without reproduction steps cannot be fixed reliably.

---

## Step 2 — Determine BUG Number

Read the `.bugs/` listing from above.
- If no bugs exist yet → this is BUG-001
- If bugs exist → increment the highest existing number (BUG-004 → BUG-005)

---

## Step 3 — Classify Severity

| Severity | Definition | Example |
|---|---|---|
| **Critical** | Core functionality broken, no workaround, blocks release | Login fails for all users, data loss, security vulnerability |
| **High** | Feature unusable, workaround exists, or affects most users | Form submits but confirmation never shows, export produces wrong data |
| **Medium** | Degraded experience, feature still works, subset of users affected | Pagination off by one, slow query on edge case |
| **Low** | Cosmetic, copy, or minor UX with no functional impact | Button misaligned, spelling error, wrong colour |

**Auto-escalation rules:**
- Any security vulnerability → Critical
- Any data corruption or loss → Critical
- Any bug breaking a committed sprint item's acceptance criteria → at least High

---

## Step 4 — Assign Owner Agent

| Bug Location | Owner Agent |
|---|---|
| UI, component, form, styling | frontend-engineer |
| API, business logic, auth, DB query | backend-engineer |
| Full-stack data flow (UI → API → DB) | full-stack-engineer |
| CI/CD, deployment, environment | devops-cloud-engineer |
| LLM, RAG, prompt, embeddings | genai-llm-engineer |
| ML model output, data pipeline | data-ml-engineer |
| Cross-cutting or unknown | hiring-manager-orchestrator investigates first |

---

## Step 5 — Assign to Sprint

| Severity | Assignment |
|---|---|
| **Critical** | Current sprint — assigned agent pauses lower-priority work |
| **High** | Current sprint — normal priority |
| **Medium** | Next sprint backlog |
| **Low** | Backlog |

Override: if fixing requires reworking an already QA-verified item → run `/change-request {project}` instead.

---

## Step 6 — Create BUG-{N}.md

Create `output/{project}/.bugs/BUG-{N}.md` using the template at `assets/bug-template.md`. Fill in all `{placeholder}` values from the bug details collected in Steps 1–5:
- `{N}` — bug number from Step 2
- `{short descriptive title}` — clear one-line description of the defect
- `{Critical / High / Medium / Low}` — severity from Step 3
- `{Functional / Security / Performance / UI / Data}` — type based on bug location
- `{project}` — project name from $ARGUMENTS
- `{dev agent}` — assigned owner from Step 4
- All reproduction steps, frequency, environment, expected/actual behaviour — from Step 1
- Sprint number — from current SPRINT.md state

---

## Step 7 — Update SPRINT.md

Add a one-line reference to the **Bug Log** table in SPRINT.md:
```
| BUG-{N} | {today} | {title} | {severity} | {agent} | {sprint} | OPEN |
```

If Critical or High in current sprint: also add to **Current Sprint** committed items:
```
| BUG-{N} | {title} [BUG] | {severity} | {agent} | OPEN |
```

If Critical, output this alert:
```
🚨 CRITICAL BUG — BUG-{N}
{agent} should pause current work and address this immediately.
All agents working on features that depend on the affected area
should be notified by hiring-manager-orchestrator.
```

Confirm: `✅ BUG-{N} created at output/{project}/.bugs/BUG-{N}.md. Severity: {severity}. Assigned: {agent}. Sprint: {sprint}.`
`Fix: /task-start {project} BUG-{N} | Verify: /bug-verify {project} BUG-{N}`
