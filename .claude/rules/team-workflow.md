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
- Feature is implemented and code-reviewed
- Unit tests are written and passing
- No known critical bugs
- API documentation is up to date
- Integration tests pass

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
| Decision | When to use |
|---|---|
| **Absorb** | Trivial/Small change, capacity available, no rework needed |
| **Defer** | Any change that doesn't block current sprint delivery |
| **Reject** | Contradicts out_of_scope, breaks deadline, or has no business justification |

Only the orchestrator + product owner can approve an Absorb decision on Medium or larger changes.

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
