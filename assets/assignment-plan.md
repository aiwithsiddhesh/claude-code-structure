# Assignment Plan: {project.display_name}

**Project**: {project.display_name}
**Type**: {project.type}
**Deadline**: {timeline.deadline}
**Security Level**: {non_functional.security_level}
**Plan Generated**: {today}

---

## Agents Assigned

| Agent | Role | Assigned Because |
|---|---|---|
| hiring-manager-orchestrator | Coordination & release | Always assigned |
| manual-functional-sdet | QA sign-off | Always assigned — mandatory release gate |
| {agent-name} | {role} | {reason from Step 3 logic} |

---

## Hard Constraints

[From `context.constraints` in project-intake.json. If empty: "No hard constraints recorded — normal delivery standards apply."]

- {constraint 1}

---

## Out of Scope

[From `requirements.out_of_scope` in project-intake.json. If empty: "No explicit out-of-scope items recorded."]

- {out-of-scope item 1}

---

## Phase 1 — Setup & Architecture

| Task | Agent | Notes |
|---|---|---|
| Validate intake and create output directory | hiring-manager-orchestrator | Done by /start-project |
| Run /design-doc for each Medium/Large/XL feature | Lead agent per feature | Must complete before Phase 2 tasks begin |
| Initialize SPRINT.md | hiring-manager-orchestrator | Done by /start-project |

---

## Phase 2 — Feature Development

| Feature | Complexity | Agent | Dependencies | Design Doc | Acceptance Criteria |
|---|---|---|---|---|---|
| {feature name} | {Small/Medium/Large/XL} | {assigned agent} | {dependencies or "none"} | {pending/done/waived} | {criteria} |

---

## Phase 3 — QA & Release

| Task | Agent | Trigger |
|---|---|---|
| Manual QA for each feature | manual-functional-sdet | After dev agent marks task READY FOR QA |
| Regression automation | automation-sdet-agent | After manual-functional-sdet marks test AUTOMATION READY |
| Release checklist | hiring-manager-orchestrator | Run /release-checklist {project.name} |

---

## Timeline Check

**Deadline**: {timeline.deadline}
**Features committed**: {count of features}
**Estimated sprints needed**: {calculated: deadline / sprint length, rounded up}
**Risk assessment**: {GREEN if capacity fits / YELLOW if tight / RED if deadline is unrealistic}

[If RED or YELLOW: "Recommend reducing scope or extending deadline before development begins. Discuss with product owner."]

---

## Next Immediate Actions

1. Run `/design-doc {feature-name}` for every Medium/Large/XL feature before any development begins
2. Run `/sprint-plan {project.name}` to commit Sprint 1 once design docs are complete
3. {agent-name}: begin with [first action specific to this project]
4. Orchestrator monitors for blockers and ensures handoffs complete between agents
