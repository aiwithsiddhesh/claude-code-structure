---
name: start-project
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

Use the template at `assets/assignment-plan.md`. Fill in all `{placeholder}` values from the intake data above:
- `{project.display_name || project.name}` — from intake `project` section
- `{project.type}` — from intake
- `{timeline.deadline}` — from intake
- `{non_functional.security_level}` — from intake, default `standard`
- Agents Assigned table — from Step 3 agent mapping
- Hard Constraints — from `context.constraints`
- Out of Scope — from `requirements.out_of_scope`
- Phase 2 rows — one row per committed feature, assigned agent from Step 3
- Timeline Check — calculated from deadline vs feature count
- Next Immediate Actions — use the agents assigned in Step 3

---

## Step 6 — Save Plan and Confirm

After producing the plan:
1. Confirm the output directory will be created: `output/{project.name}/`
2. Remind the user to run `/design-doc [feature-name]` before development begins on each feature
3. Save key project facts to agent memory for use in future sessions

---

## Step 7 — Initialize SPRINT.md

Create `output/{project.name}/SPRINT.md` using the template at `assets/sprint-md-template.md`. Fill in all `{placeholder}` values from the intake data:
- `{project.name}` and `{project.display_name or project.name}` — from intake
- `{today}` — current date
- `{timeline.deadline}` — from intake
- `{calculated: deadline / sprint length, rounded up}` — compute from deadline
- Backlog rows — one row per item in `requirements.features[]`

Confirm: `✅ SPRINT.md initialized at output/{project.name}/SPRINT.md. {count} backlog items loaded from intake. Run /sprint-plan {project.name} to begin Sprint 1.`

