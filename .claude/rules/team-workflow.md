# Team Workflow Rules

## Project Lifecycle

```
Intake → Planning → Development → QA → Release → Retrospective
```

Quality gates must pass before advancing each stage.

## Handoff Protocol

Every handoff between agents MUST include:
- **Task Owner** — Who is receiving this work
- **Context** — Why this task exists, business goal
- **Completed Work** — What has been done
- **Pending Work** — What still needs to be done
- **Risks & Blockers** — Known issues or unknowns
- **Acceptance Criteria** — How to verify this is done correctly
- **Next Actions** — Specific immediate steps for the receiver

## Quality Gates

### Before Development Starts
- Requirements are documented, testable, and unambiguous
- Acceptance criteria are defined and measurable
- API contracts are agreed upon between frontend and backend
- DoR (Definition of Ready) is confirmed by the Business Analyst

### Before QA Starts
- Feature is implemented and `/code-review` APPROVED result is recorded in TASK.md Completion entry
- Unit tests are written and passing
- No known critical bugs
- API documentation is up to date
- Integration tests pass

### QA Handoff — Manual to Automation

`manual-functional-sdet` manages the transition to `automation-sdet-agent` directly, without requiring an orchestrator assignment. A test case is **AUTOMATION READY** when ALL of the following are true:

1. The feature has passed manual QA at least once (TASK.md status is `DONE` or the bug shows `VERIFIED`)
2. The test case has clear, documented reproduction steps with no ambiguity
3. The test case passed on two consecutive manual runs without changes to steps
4. The acceptance criteria the test validates have not changed in the current sprint (no open change requests affecting this feature)
5. The feature is not scheduled for significant rework in the next sprint

Automation is **NOT** appropriate for:
- Features with task status still `IN PROGRESS`, `BLOCKED`, or `REWORK`
- Test cases that depend on exploratory steps or human judgment
- One-off validation tests that will not recur
- Features with open change requests that could alter the acceptance criteria

`manual-functional-sdet` marks qualifying test cases as `AUTOMATION READY` in their test execution notes. `automation-sdet-agent` reads this signal and picks up those test cases without waiting for orchestrator assignment.

### Before Release
- QA (manual-functional-sdet) has signed off
- Automated regression tests pass (automation-sdet-agent)
- Security review completed (if applicable)
- Performance benchmarks met
- Release notes drafted
- Rollback plan defined

## Escalation Rules

| Trigger | Escalate To |
|---|---|
| Unclear requirements | business-analyst-agent |
| Scope or priority conflict | Product Owner |
| Timeline or resource risk | hiring-manager-orchestrator |
| Technical blocker no agent can resolve | hiring-manager-orchestrator with options |

## Blocker Log Format

```
Description: [What is blocked]
Impact: High / Medium / Low
Owner: [Who is resolving]
Root Cause: [Why it's blocked]
Resolution: [Proposed fix]
Target Date: [When it will be resolved]
```

## Sprint Rules

### Sprint Cycle
Every project runs in weekly sprints. The cycle is:

```
/start-project  →  /sprint-plan  →  [agents work]  →  /sprint-review  →  /sprint-plan  → ...
                                           ↓                    ↓
                                   /change-request         /bug-triage
                                   (if req changes)        (if bugs found)
```

### Mid-Sprint Rules
- No new work enters an active sprint without a `/change-request` decision
- `SPRINT.md` is the single source of truth — all agents read it before acting
- Incomplete sprint items return to the backlog at review — never auto-carry silently
- The orchestrator must be notified of any bug before an agent self-assigns a fix

### Change Control

**Who can request a change:** Anyone — any agent, product owner, or stakeholder. All requests go through `/change-request`. Requests are not filtered at entry.

**Who has authority to decide:**

| Change Size | Authority | Timeline |
|---|---|---|
| Trivial / Small | Orchestrator alone | Same session — decide immediately |
| Medium | Orchestrator + product owner must agree | If product owner unavailable this sprint → auto-defer |
| Large / Breaking | Orchestrator + product owner required | Sprint may need to pause and replan via `/sprint-plan` after decision |
| Reject (any size) | Orchestrator alone | Must record reason in SPRINT.md in enough detail that the requestor understands why |

**Response time SLA:**

| Impact Level | Definition | SLA |
|---|---|---|
| Critical | Would block current sprint delivery | Decision required before any agent continues on affected tasks — same session |
| High | Affects 2+ committed sprint items | Decision required within current sprint — cannot carry into next sprint undecided |
| Medium | Affects 1 committed item | Decision by end of current sprint |
| Low | Does not affect any committed item | Can be deferred to next `/sprint-plan` without a formal decision, but must be recorded in backlog |

**What happens to in-progress work during review:**

| Change Size | In-Progress Work Rule |
|---|---|
| Trivial / Small | Work continues uninterrupted |
| Medium | Affected tasks move to `ON_HOLD` until decision is made |
| Large / Breaking | All dependent tasks move to `ON_HOLD` immediately |

**Decision outcomes:**

| Decision | When to use |
|---|---|
| **Absorb** | Trivial/Small change, capacity available, does not require reworking completed items |
| **Defer** | Any change that does not block current sprint delivery |
| **Reject** | Contradicts `out_of_scope`, breaks deadline, or has no business justification — reason must be recorded |

### Bug Severity Sprint Rules
| Severity | Sprint Assignment |
|---|---|
| Critical | Current sprint — pause lower-priority work |
| High | Current sprint — normal priority |
| Medium | Next sprint backlog |
| Low | Backlog |

Security vulnerabilities and data loss bugs are always Critical regardless of scope.

### Artifact Authority
When multiple documents exist (design doc, handoff, status report, SPRINT.md), **SPRINT.md is always the most current state**. If SPRINT.md contradicts a design doc or handoff, SPRINT.md wins. Stale artifacts should be noted in SPRINT.md but not deleted.

## Task State Machine

Every sprint task has a **Status** field in its TASK.md. No state may be skipped.

```
IN PROGRESS ──→ BLOCKED        (agent hits an external dependency it cannot resolve)
IN PROGRESS ──→ REWORK         (manual-functional-sdet rejects the task after QA)
IN PROGRESS ──→ READY FOR QA   (all steps done, /task-checkpoint completion entry written)
BLOCKED     ──→ IN PROGRESS    (blocker is resolved — agent resumes)
REWORK      ──→ IN PROGRESS    (agent begins fixing QA-identified defects)
ON_HOLD     ──→ IN PROGRESS    (orchestrator releases the hold)
READY FOR QA──→ DONE           (manual-functional-sdet signs off)
READY FOR QA──→ REWORK         (manual-functional-sdet rejects after reviewing)
```

| State | Set By | Meaning |
|---|---|---|
| `IN PROGRESS` | Dev agent (via `/task-start`) | Agent is actively working on the task |
| `BLOCKED` | Dev agent | Task cannot proceed — waiting on an external dependency, decision, credential, or another task to complete. Agent must document the blocker and owner before stopping work. |
| `REWORK` | `manual-functional-sdet` | Task was reviewed and rejected. Specific defects must be fixed before resubmission. Agent can work on it immediately — it is not blocked, just targeted for fixes. |
| `ON_HOLD` | `hiring-manager-orchestrator` only | Deliberately paused by the orchestrator. Agent must NOT work on it. Only the orchestrator can release this hold. |
| `READY FOR QA` | Dev agent (via `/task-checkpoint` completion flow) | All steps complete, completion entry written, awaiting `manual-functional-sdet` validation. |
| `DONE` | `manual-functional-sdet` | QA has signed off. All acceptance criteria validated. |

**Authority rules:**
- Dev agents set `BLOCKED` themselves — they know when they are stuck.
- `manual-functional-sdet` sets `REWORK` — never the dev agent.
- Only `hiring-manager-orchestrator` sets `ON_HOLD` and only the orchestrator releases it.
- A task BLOCKED for more than one full sprint is an escalation trigger for the orchestrator.

## Task Checkpoint Quality Gate

A sprint task moves to `DONE` only when:
1. All steps in `output/{project}/.tasks/{task-id}.md` are checked off
2. `/task-checkpoint` has written a Completion entry (not just a mid-session checkpoint)
3. `/code-review` has been run and the APPROVED result is recorded in the Completion entry
4. `manual-functional-sdet` has read the TASK.md completion entry and validated against acceptance criteria
5. `manual-functional-sdet` has updated the task status to `DONE` in TASK.md

An agent self-reporting "done" without a TASK.md completion entry is not done.
A `/sprint-review` must verify TASK.md completion status for every committed item before closing a sprint. BLOCKED and ON_HOLD tasks must be explicitly decided by the orchestrator — they are not silently carried forward.

### Checkpoint Frequency Rule
Any session working on a sprint task that ends without completing the task **must** run `/task-checkpoint` before the session closes. A task with no checkpoint after 2+ sessions is a signal the task is too large — flag to orchestrator for splitting.

## Bug Lifecycle States

Every bug moves through these states in `BUG-{N}.md`. No skipping.

```
OPEN → IN FIX → READY FOR VERIFICATION → VERIFIED → CLOSED
              ↑                        |
              └──── REJECTED ──────────┘
```

| State | Set by | Meaning |
|---|---|---|
| `OPEN` | `/bug-triage` | Bug documented, assigned, not yet being fixed |
| `IN FIX` | Dev agent (via `/task-start`) | Dev agent is actively working the fix |
| `READY FOR VERIFICATION` | Dev agent | Fix complete, awaiting manual-functional-sdet |
| `REJECTED` | `/bug-verify` | Fix failed verification, returned to dev agent |
| `VERIFIED` | `/bug-verify` | Fix confirmed, awaiting regression test |
| `CLOSED` | automation-sdet-agent | Regression test written and passing in CI |

**A bug is not fixed until CLOSED. VERIFIED means the fix works but recurrence is not yet prevented.**

Bug files live at `output/{project}/.bugs/BUG-{N}.md`.
SPRINT.md Bug Log holds one-line references only — always read BUG-{N}.md for full state.
