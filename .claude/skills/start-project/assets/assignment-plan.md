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

### Phase 1 — Requirements & Design
| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| Requirements clarification | business-analyst-agent | features[] from intake | User stories with acceptance criteria | DoR checklist passed |
| Technical design (per feature) | [lead engineer agent] | User stories | /design-doc completed | Design approved |

### Phase 2 — Development
| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| [feature 1] | [agent] | Design doc, acceptance criteria | Code in output/{name}/ | Code reviewed |
| [feature N] | [agent] | ... | ... | ... |
| Infra + CI/CD setup | devops-cloud-engineer | Stack config from intake | Pipeline, environments | Staging deploy works |

### Phase 3 — QA & Release
| Step | Agent | Input | Output | Gate |
|---|---|---|---|---|
| Manual QA | manual-functional-sdet | All acceptance criteria | Test results, QA sign-off | All criteria pass |
| Automation suite | automation-sdet-agent | Stable test cases | CI test suite | Tests pass in pipeline |
| Release | hiring-manager-orchestrator | /release-checklist output | Production deployment | Release checklist GO |

---

## Quality Gates

### DoR — Definition of Ready (BA → Dev handoff)
- [ ] All user stories written with Given/When/Then acceptance criteria
- [ ] API contracts defined (if frontend + backend)
- [ ] No open critical questions

### DoD — Definition of Done (Dev → QA handoff)
- [ ] Feature implemented and code-reviewed
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
| `/design-doc [feature]` | Before development on any non-trivial feature |
| `/handoff [from] to [to]` | At every agent transition |
| `/status-report {project.name}` | Weekly or on request |
| `/release-checklist {project.name}` | Before any production deployment |
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

1. business-analyst-agent: Convert features[] into user stories with acceptance criteria → use `/handoff business-analyst-agent to [lead dev agent]` when done
2. [lead dev agent]: Review user stories, run `/design-doc` for each major feature
3. devops-cloud-engineer: Set up {stack.hosting} environment and CI/CD pipeline (can run in parallel with requirements)
4. hiring-manager-orchestrator: Schedule first status check at 25% of timeline

---
*Generated from project-intake.json — commit this file to record the agreed delivery plan.*
