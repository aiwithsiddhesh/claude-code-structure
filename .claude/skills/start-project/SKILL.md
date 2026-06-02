---
description: Kick off a new project by reading project-intake.json. Run after filling in project-intake.json. The skill validates the intake, maps stack to agents, sets the output directory, and produces a full delivery plan.
disable-model-invocation: true
---

# Project Kickoff

## Intake Data

```json
!`cat project-intake.json 2>/dev/null || echo '{"_error": "project-intake.json not found. Copy the template from the repo root and fill it in before running this skill."}'`
```

---

You are the **hiring-manager-orchestrator**. Work through each step below using the intake data above.

---

## Step 1 — Validate Intake

Check every required field. If any are invalid, stop and list ALL issues before proceeding.

**Required field checks:**
- `project.name` — must be non-empty, kebab-case (lowercase, hyphens only, no spaces)
- `project.description` — must be non-empty (minimum 20 characters)
- `project.type` — must be one of: `web-app`, `api`, `data-pipeline`, `mobile`, `ai-feature`, `internal-tool`
- `requirements.features` — must have at least one item
- `timeline.deadline` — must be non-empty

**Conditional checks:**
- If `stack.ai_ml` is `true` → `stack.ai_ml_type` must be set
- If `context.existing_codebase` is `true` → `context.codebase_notes` must be non-empty
- If `team.agents_needed` and `team.skip_agents` both contain the same agent → flag as conflict
- If `team.skip_agents` contains `manual-functional-sdet` → hard error, do not proceed (see Step 3 for error message)

If validation fails:
```
❌ INTAKE VALIDATION FAILED

The following fields need attention before the project can start:
- [field]: [what is wrong and how to fix it]

Edit project-intake.json and run /start-project again.
```

---

## Step 2 — Resolve Output Directory

Set the output directory:
- If `output.dir` is empty → use `output/{project.name}/`
- If `output.dir` is set → use that value

**Announce clearly:**
```
📁 All project files will be written to: output/{project.name}/
```

Every agent working on this project MUST create and write inside this directory per `coding-standards.md`.

---

## Step 3 — Map Stack to Agents

Use the intake data to determine which agents are needed. Apply this logic:

**Always assigned:**
- `hiring-manager-orchestrator` — coordination (that's you)
- `manual-functional-sdet` — QA sign-off before any release

**Assign based on stack + type:**

| Condition | Assign |
|---|---|
| `requirements.features` is not empty | `business-analyst-agent` |
| `stack.frontend` is set and not `none` | `frontend-engineer` |
| `stack.backend` is set and not `none` | `backend-engineer` |
| Both `stack.frontend` AND `stack.backend` are set AND `project.type` is `web-app` or `internal-tool` | Consider `full-stack-engineer` instead of separate frontend + backend (see note below) |
| `stack.database` is set and not `none` | `backend-engineer` handles DB (already assigned above) |
| `stack.hosting` is set and not `none` or `undecided` | `devops-cloud-engineer` |
| `stack.ai_ml` is `true` AND `stack.ai_ml_type` is `llm-rag` or `both` | `genai-llm-engineer` |
| `stack.ai_ml` is `true` AND `stack.ai_ml_type` is `ml-model` or `both` | `data-ml-engineer` |
| Feature list has 3+ features that are stable/well-defined | `automation-sdet-agent` |

**full-stack vs separate frontend + backend rule:**
- Use `full-stack-engineer` when features are tightly coupled (UI + API + DB in one flow) and work is sequential
- Use separate `frontend-engineer` + `backend-engineer` when work can be parallelized (API contract agreed upfront)
- When unsure → use `full-stack-engineer` for projects under 3 weeks, split for longer

**Apply overrides from intake:**
- `team.agents_needed` → force-add these even if not triggered above

**QA gate protection — check BEFORE applying skip_agents:**
If `manual-functional-sdet` appears in `team.skip_agents`, stop immediately and output:

```
❌ INVALID INTAKE: manual-functional-sdet cannot be removed.

manual-functional-sdet is a mandatory release gate defined in
.claude/rules/agent-coordination.md. No project may release without
QA sign-off regardless of team size, timeline, or MVP scope.

If you want to reduce QA scope, set a shorter feature list — do not
remove the QA agent. Edit project-intake.json, remove
manual-functional-sdet from skip_agents, and run /start-project again.
```

- `team.skip_agents` → remove these even if triggered above (after QA gate check passes)

---

## Step 4 — Apply Constraints and Context

**Constraints from `context.constraints`** → list these as hard rules in the assignment plan. Every agent must acknowledge them.

**Existing codebase from `context.existing_codebase`:**
- If `true` → add instruction to ALL agents: "Read and understand the existing codebase before writing any new code. Notes: {context.codebase_notes}"

**Reference docs from `context.reference_docs`:**
- If not empty → add instruction: "Read these documents before starting: {list}"

**Security level from `requirements.non_functional.security_level`:**
- `standard` → normal `security.md` rules apply
- `high` → add to plan: mandatory security review on every PR, penetration test required before release, auth implementation requires explicit security sign-off, security checklist from `security.md` is blocking (not advisory)

---

## Step 5 — Produce the Assignment Plan

Output this document:

```markdown
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
```

---

## Step 6 — Save Plan and Confirm

After producing the plan:
1. Confirm the output directory will be created: `output/{project.name}/`
2. Remind the user to run `/design-doc [feature-name]` before development begins on each feature
3. Save key project facts to agent memory for use in future sessions

---

## Step 7 — Initialize SPRINT.md

Create `output/{project.name}/SPRINT.md` with the following structure. Fill in all values from the intake data:

```markdown
# SPRINT.md — {project.name}
<!-- Single source of truth for sprint state. All agents must read this before making decisions. -->

## Project

| Field | Value |
|---|---|
| **Name** | {project.display_name or project.name} |
| **Start Date** | {today} |
| **Deadline** | {timeline.deadline} |
| **Sprint Length** | 1 week |
| **Total Sprints** | {calculated: deadline / sprint length, rounded up} |
| **Current Sprint** | 0 — not started |

---

## Current Sprint

> No sprint active. Run `/sprint-plan {project.name}` to commit Sprint 1.

---

## Backlog

All features from project intake. Items are pulled into sprints via `/sprint-plan`.
Bugs and change requests are appended here automatically by `/bug-triage` and `/change-request`.

| ID | Item | Source | Priority | Notes |
|---|---|---|---|---|
| B-1 | {feature 1 from intake} | intake | - | |
| B-2 | {feature 2 from intake} | intake | - | |
| B-N | {feature N from intake} | intake | - | |

---

## Change Requests

| ID | Date | Description | Size | Decision | Sprint |
|---|---|---|---|---|---|
| — | | | | | |

---

## Bug Log

| ID | Date | Description | Severity | Agent | Sprint | Status |
|---|---|---|---|---|---|---|
| — | | | | | | |

---

## Sprint History

| Sprint | Dates | Committed | Completed | Carry-forward | Notes |
|---|---|---|---|---|---|
| — | | | | | |
```

Confirm: `✅ SPRINT.md initialized at output/{project.name}/SPRINT.md. {count} backlog items loaded from intake. Run /sprint-plan {project.name} to begin Sprint 1.`

