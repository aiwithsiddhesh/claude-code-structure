# Project Assignment Plan

**Project**: {project.display_name || project.name}
**Type**: {project.type}
**Deadline**: {timeline.deadline}
**Output Directory**: output/{project.name}/
**Security Level**: {non_functional.security_level}

---

## Agents Assigned

| Agent | Role on This Project | Key Tasks |
|---|---|---|
| hiring-manager-orchestrator | Coordinator | Track progress, resolve blockers, enforce quality gates, produce delivery summary |
| [agent] | [role] | [specific tasks derived from features list] |

## Hard Constraints (all agents must follow)
{list from context.constraints, or "None specified" if empty}

## Out of Scope (do not build)
{list from requirements.out_of_scope, or "None specified" if empty}

---

## Delivery Sequence

Development cannot begin until BA readiness and design/ambiguity gates have passed. No item enters a sprint without acceptance criteria or an explicit waiver.

### Phase 1 — Requirements & BA Readiness

BA readiness must complete before sprint planning. Sprint planning must not commit items without acceptance criteria.

| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| Requirements clarification | business-analyst-agent | features[] from intake | User stories with acceptance criteria, assumptions, open questions, priority, size, DoR status | DoR checklist passed — all items show BA Status: ready in SPRINT.md |
| Ambiguity classification | business-analyst-agent | User stories | Ambiguity rating (Low/Medium/High) per item recorded in SPRINT.md backlog | All Medium/High ambiguity items have resolution plan before sprint commit |

### Phase 2 — Design

Design/ambiguity gate runs before development, not just before individual tasks.

| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| Technical design (per Medium/Large/XL feature) | [lead engineer agent] | User stories, acceptance criteria | /design-doc completed, SPRINT.md updated to design-doc: done | Section 9 Approval Record signed by non-author; SPRINT.md shows design-doc: done |
| Ambiguity review (all items, regardless of size) | [lead engineer agent] | User stories | Ambiguity resolved or waiver recorded | No item enters sprint with Ambiguity Medium/High unresolved |

### Phase 3 — Development

| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| [feature 1] | [agent] | Design doc, acceptance criteria | Code in output/{name}/ | Code reviewed — APPROVED recorded in TASK.md |
| [feature N] | [agent] | ... | ... | ... |
| Infra + CI/CD setup | devops-cloud-engineer | Stack config from intake | Pipeline, environments | Staging deploy works |

### Phase 4 — QA & Release

| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| Manual QA | manual-functional-sdet | All acceptance criteria, TASK.md completion entries | Test results, QA sign-off | All criteria pass; TASK.md status = DONE |
| Automation suite | automation-sdet-agent | AUTOMATION READY test cases from manual QA | CI test suite | Tests pass in pipeline |
| Release | hiring-manager-orchestrator | /release-checklist output (reads TASK.md + BUG-N.md + SPRINT.md) | Production deployment | Release checklist GO |

---

## Quality Gate Map

| Gate | Skill / Point | Failure Class Caught |
|---|---|---|
| Intake validation | `/start-project` Step 1 | Missing/malformed fields, invalid stack config |
| BA readiness | business-analyst-agent → SPRINT.md BA Status | Vague features, missing acceptance criteria before sprint commit |
| Sprint commit readiness | `/sprint-plan` Step 3 | Items without acceptance criteria or unresolved Medium/High ambiguity enter sprint |
| Ambiguity gate | `/sprint-plan` Step 3 + `/task-start` Step 2 | Small items with Medium/High ambiguity slip through without review |
| Design gate | `/design-doc` → SPRINT.md design-doc column | Missing API/data/security decisions, unapproved architecture |
| DoR gate | `/task-start` Step 2 | Vague criteria, oversized stories, unresolved open questions reach development |
| Code review | `/code-review` | Correctness, security, quality, test coverage gaps |
| QA gate | `manual-functional-sdet` | Feature doesn't meet acceptance criteria, regressions |
| Bug verification | `/bug-verify` | Fix doesn't work or breaks adjacent behaviour |
| Regression gate | `automation-sdet-agent` | Bug recurs in future; no CI coverage |
| Sprint review | `/sprint-review` | Status drift, incomplete items silently carried forward |
| Release gate | `/release-checklist` | Unreleased bugs, missing QA sign-off, infra not ready |

---

## Quality Gates

### DoR — Definition of Ready (BA → Dev handoff)
- [ ] All user stories written with Given/When/Then acceptance criteria
- [ ] All items have Ambiguity rating in SPRINT.md backlog
- [ ] API contracts defined (if frontend + backend)
- [ ] No open critical questions — all `[ ]` items resolved or deferred with an owner
- [ ] BA Status = `ready` for all committed items (no `needs-BA` items in sprint)

### DoD — Definition of Done (Dev → QA handoff)
- [ ] Feature implemented and code-reviewed (APPROVED in TASK.md Completion entry)
- [ ] Tests written and passing
- [ ] No critical/high bugs open
- [ ] API documentation updated

### QA Sign-Off (QA → Release)
- [ ] All acceptance criteria tested and passing
- [ ] Regression tests pass
- [ ] No open Critical or High bugs
{if security_level == high: - [ ] Security review completed\n- [ ] Penetration test passed}

---

## Skills Available on This Project

| Command | When to use |
|---|---|
| `/design-doc [feature]` | Required for Medium/Large/XL items; required for any item with Ambiguity Medium/High; updates SPRINT.md on completion |
| `/handoff [from] to [to]` | At every agent transition |
| `/status-report {project.name}` | Weekly or on request — reads SPRINT.md + TASK.md + BUG-N.md |
| `/release-checklist {project.name}` | Before any production deployment — reads SPRINT.md + TASK.md + BUG-N.md |
| `/retrospective {project.name}` | After delivery |

---

## Timeline Check

Deadline: {timeline.deadline}
Features: {count of requirements.features} features
Agents: {count of assigned agents}

{If timeline appears tight for the scope: flag as AT RISK with reasoning}
{If timeline is realistic: confirm as FEASIBLE}

Milestones:
{list milestones if provided, or "None specified"}

---

## Next Immediate Actions

1. business-analyst-agent: Convert features[] into user stories with acceptance criteria, assumptions, open questions, priority, size, and DoR status → update SPRINT.md backlog BA Status to `ready` for each item → use `/handoff business-analyst-agent to [lead dev agent]` when done
2. [lead dev agent]: Review user stories, run `/design-doc` for each Medium/Large/XL feature and each item with Ambiguity Medium/High
3. devops-cloud-engineer: Set up {stack.hosting} environment and CI/CD pipeline (can run in parallel with requirements)
4. hiring-manager-orchestrator: Do not run `/sprint-plan` until all committed items have BA Status = `ready`

---
*Generated from project-intake.json — commit this file to record the agreed delivery plan.*
