# AI Software Delivery Team Framework

This repository defines a reusable AI software delivery framework for planning, building, testing, and releasing software projects through specialized Claude agents. It is designed to turn a structured project intake into an agent assignment plan, project backlog, sprint workflow, quality gates, and generated implementation files.

The framework is documentation-first: agent definitions, operating rules, and reusable skills are built before sample applications. Generated project code belongs under `output/{project-name}/`, not in the repository root.

## What This Repository Contains

- A structured project intake file: `project-intake.json`
- A JSON schema for intake validation and VS Code autocomplete: `project-intake.schema.json`
- Claude agent definitions in `.claude/agents/`
- Always-on and path-scoped operating rules in `.claude/rules/`
- Reusable slash-command skills in `.claude/skills/`
- VS Code schema wiring in `.vscode/settings.json`
- Claude-facing repository instructions in `CLAUDE.md`

## Core Idea

The framework acts like an AI delivery operating system. A project starts with an intake document, then the orchestrator maps the project to the right specialist agents, creates a delivery plan, initializes sprint tracking, and enforces handoffs and quality gates throughout delivery.

The lifecycle is:

```text
Intake -> Planning -> Development -> QA -> Release -> Retrospective
```

Each stage has a clear owner, expected inputs, required outputs, and acceptance criteria.

## Quick Start

1. Open `project-intake.json`.
2. Fill in the project identity, stack, features, non-functional requirements, timeline, and any constraints.
3. Keep the `$schema` property pointing to `./project-intake.schema.json`.
4. In Claude, run:

```text
/start-project
```

5. Review the generated assignment plan.
6. Continue delivery through the sprint workflow:

```text
/sprint-plan
[assigned agents complete work]
/sprint-review
/release-checklist
/retrospective
```

## Project Intake

`project-intake.json` is the entry point for every new project. The schema requires these top-level sections:

| Section | Purpose |
|---|---|
| `project` | Project name, display name, description, and category |
| `stack` | Frontend, backend, database, auth, hosting, and AI/ML choices |
| `requirements` | Features, exclusions, expected scale, performance, and security level |
| `timeline` | Deadline and optional milestones |
| `team` | Optional manual agent include/exclude overrides |
| `context` | Existing-codebase notes, reference docs, and hard constraints |
| `output` | Optional generated file output directory override |

### Required Project Fields

| Field | Notes |
|---|---|
| `project.name` | Kebab-case identifier. Used for `output/{project.name}/` |
| `project.description` | Two to three sentences describing the goal and users |
| `project.type` | One of `web-app`, `api`, `data-pipeline`, `mobile`, `ai-feature`, or `internal-tool` |
| `requirements.features` | At least one specific feature |
| `timeline.deadline` | ISO date, relative estimate, or quarter |

### Supported Stack Values

| Area | Values |
|---|---|
| Frontend | `react`, `next.js`, `vue`, `svelte`, `react-native`, `none` |
| Backend | `node.js-express`, `node.js-fastify`, `python-fastapi`, `python-django`, `python-flask`, `go`, `none` |
| Database | `postgresql`, `mysql`, `mongodb`, `sqlite`, `redis`, `supabase`, `none` |
| Auth | `jwt`, `oauth2`, `session-cookie`, `api-key`, `supabase-auth`, `auth0`, `none` |
| Hosting | `aws`, `gcp`, `azure`, `vercel`, `railway`, `fly.io`, `self-hosted`, `undecided` |
| AI/ML Type | `llm-rag`, `ml-model`, `both` |

Leave optional string values empty when undecided. Set `stack.ai_ml` to `true` only when the project requires LLMs, RAG, embeddings, chatbots, ML models, or data pipelines.

## Repository Structure

```text
.
+-- .claude/
|   +-- agents/                 # Specialist agent definitions
|   +-- rules/                  # Always-on and path-scoped operating rules
|   +-- skills/                 # Reusable slash-command workflows
+-- .vscode/
|   +-- settings.json           # JSON schema binding for project-intake.json
+-- CLAUDE.md                   # Claude-facing repository instructions
+-- CLAUDE.local.md             # Local personal notes, not intended for git
+-- project-intake.json         # Project kickoff input
+-- project-intake.schema.json  # Intake validation schema
+-- README.md                   # Human-facing project documentation
```

Generated implementation work should be placed in:

```text
output/{project-name}/
```

The root of this repository is reserved for framework files, not generated application code.

## Agents

The framework includes 10 specialist agents across planning, engineering, infrastructure, AI/ML, and QA.

| Agent | Primary Responsibility |
|---|---|
| `hiring-manager-orchestrator` | Coordinates projects, selects agents, resolves blockers, manages delivery flow |
| `business-analyst-agent` | Clarifies requirements, user stories, acceptance criteria, MVP scope, risks |
| `frontend-engineer` | Builds and debugs user interfaces, forms, accessibility, responsive behavior |
| `backend-engineer` | Designs APIs, databases, auth, backend logic, integrations, validation |
| `full-stack-engineer` | Handles tightly coupled frontend/backend features that should not be split |
| `devops-cloud-engineer` | Handles infrastructure, CI/CD, containers, deployment, monitoring |
| `genai-llm-engineer` | Handles LLMs, RAG, embeddings, vector databases, prompt engineering |
| `data-ml-engineer` | Handles ETL, data pipelines, ML models, feature engineering, evaluation |
| `manual-functional-sdet` | Performs manual QA, exploratory testing, regression checks, release sign-off |
| `automation-sdet-agent` | Builds repeatable automated tests for UI, API, regression, and CI/CD |

Use `/agent-registry` when deciding which agent should own a task.

## Skills And Commands

Skills live in `.claude/skills/` and are invoked as slash commands in Claude.

| Command | Purpose |
|---|---|
| `/agent-registry` | Show the full agent reference and assignment guidance |
| `/start-project` | Read `project-intake.json`, validate it, assign agents, create a delivery plan |
| `/design-doc [feature]` | Create a technical design document before non-trivial development |
| `/handoff [from] to [to]` | Generate a structured handoff between agents |
| `/sprint-plan [project]` | Plan and commit the next sprint from backlog items |
| `/sprint-review [project]` | Close the current sprint and carry forward incomplete work |
| `/change-request [project]` | Assess and decide a mid-sprint scope or requirement change |
| `/bug-triage [project]` | Classify a defect, assign ownership, and log it in sprint tracking |
| `/status-report [project]` | Summarize project progress, blockers, risks, and next actions |
| `/release-checklist [name]` | Verify quality gates before production release |
| `/retrospective [project]` | Capture lessons learned and process improvements after delivery |

## Operating Rules

Rules live in `.claude/rules/` and define how agents should work.

| Rule File | Scope |
|---|---|
| `team-workflow.md` | Lifecycle, handoffs, quality gates, escalation, sprint rules |
| `agent-coordination.md` | Assignment matrix, parallel vs sequential work, QA timing |
| `coding-standards.md` | Output directory, naming, formatting, comments, code quality |
| `git-workflow.md` | Branch names, conventional commits, PR expectations, release tags |
| `security.md` | Secrets, validation, auth, headers, dependency security, logging |
| `frontend-patterns.md` | React, TypeScript, component structure, state, forms, accessibility |
| `api-patterns.md` | REST URL patterns, response envelopes, pagination, validation, auth |
| `database.md` | Migrations, schema conventions, parameterized queries, ORM patterns |

## Delivery Workflow

### 1. Intake

Complete `project-intake.json` with enough detail for the business analyst and orchestrator to understand the project. Features should be specific enough to become user stories.

### 2. Assignment Planning

Run `/start-project`. The orchestrator validates the intake, resolves the output directory, maps stack choices to agents, applies constraints, and creates a project assignment plan.

### 3. Requirements And Design

The business analyst turns features into user stories and acceptance criteria. For non-trivial features, run `/design-doc` before implementation starts.

### 4. Development

Assigned engineering agents build inside `output/{project-name}/`. Agents follow stack-specific rules, security standards, and coding standards.

### 5. Handoff

Every transition between agents uses `/handoff`. A handoff must include context, completed work, pending work, risks, acceptance criteria, and next actions.

### 6. QA

Manual QA validates behavior against acceptance criteria. Automation QA builds repeatable regression coverage once test cases are stable.

### 7. Release

Run `/release-checklist` before deployment. Release requires completed development, passing tests, documentation, deployment readiness, security review where applicable, and rollback planning.

### 8. Retrospective

Run `/retrospective` after delivery or sprint closure to capture lessons, agent performance notes, metrics, and framework improvements.

## Quality Gates

### Before Development

- Requirements are clear enough to implement.
- Acceptance criteria are written.
- Scope and out-of-scope items are explicit.
- Dependencies and constraints are known.
- Required reference docs are available.

### Before QA

- Feature implementation is complete.
- Relevant unit or integration tests are present.
- Known bugs are documented.
- Handoff includes testable acceptance criteria.

### Before Release

- Manual QA has signed off.
- Automated regression tests pass when applicable.
- Security rules are satisfied.
- Deployment and rollback steps are documented.
- Open blockers are resolved or explicitly accepted.

## Sprint Tracking

`/start-project` initializes `SPRINT.md` for the generated project. During active delivery, `SPRINT.md` is the source of truth for:

- Current sprint scope
- Backlog
- Change requests
- Bug log
- Sprint history
- Project status

If another artifact conflicts with `SPRINT.md`, treat `SPRINT.md` as the current state.

## Output Directory Rule

All generated project files must go under:

```text
output/{project-name}/
```

Examples:

```text
output/customer-portal/
output/analytics-dashboard/
output/auth-service/
```

Do not place generated application source files in the repository root. The root stores the framework itself.

## Working With Existing Codebases

Set `context.existing_codebase` to `true` when agents are extending or integrating with an existing application. Then fill in:

- `context.codebase_notes` with the stack, architecture, important directories, and known constraints
- `context.reference_docs` with files agents should read first
- `context.constraints` with hard rules that must be followed

When this flag is enabled, agents should inspect existing code before writing new files.

## Customizing The Framework

### Add A New Agent

Create:

```text
.claude/agents/{agent-name}.md
```

Include frontmatter for `name`, `description`, `model`, and `color`, then define responsibilities, inputs, outputs, and definition of done.

### Add A New Rule

Create:

```text
.claude/rules/{topic}.md
```

Use always-on rules for global behavior. Use path-scoped rules when the rule only applies to certain file patterns or technology areas.

### Add A New Skill

Create:

```text
.claude/skills/{skill-name}/SKILL.md
```

Include a description and a structured workflow. Skills should produce consistent artifacts and update the correct project tracking files.

### Extend The Intake Schema

Update both:

```text
project-intake.schema.json
project-intake.json
```

Keep schema descriptions clear so VS Code autocomplete remains useful.

## Security Baseline

Every agent must follow `.claude/rules/security.md`. Core requirements include:

- Never commit secrets, API keys, passwords, tokens, or credentials.
- Validate all user input server-side.
- Use parameterized database queries.
- Enforce authentication and authorization on protected routes.
- Set appropriate HTTP security headers.
- Keep dependencies patched and audited.
- Avoid logging sensitive data.

For projects handling PII, financial data, healthcare data, or regulated workflows, set:

```json
"security_level": "high"
```

## Git Workflow

The framework expects:

- One feature or fix per branch.
- Conventional commit messages.
- Pull requests with clear what, why, how, and testing sections.
- No self-merges on protected branches.
- Squash merges for feature and fix branches.
- Release tags using semantic versioning.

See `.claude/rules/git-workflow.md` for the full policy.

## Troubleshooting

| Problem | Fix |
|---|---|
| VS Code does not autocomplete `project-intake.json` | Confirm `.vscode/settings.json` points to `./project-intake.schema.json` |
| `/start-project` reports missing fields | Fill all required schema fields, especially `project.name`, `project.description`, `project.type`, `requirements.features`, and `timeline.deadline` |
| Generated files appear in the repo root | Move them under `output/{project-name}/` and reinforce the output directory rule |
| Wrong agents are assigned | Use `team.agents_needed` or `team.skip_agents` in `project-intake.json` |
| Scope keeps expanding | Add explicit exclusions to `requirements.out_of_scope` and use `/change-request` for mid-sprint changes |
| QA starts too late | Engage `manual-functional-sdet` once acceptance criteria are stable and `automation-sdet-agent` when regression cases are repeatable |

## Maintenance Checklist

Use this checklist when changing the framework:

- Agent descriptions are updated in `.claude/agents/`.
- Command behavior is updated in `.claude/skills/`.
- Always-on standards are updated in `.claude/rules/`.
- Intake changes are reflected in both `project-intake.json` and `project-intake.schema.json`.
- `README.md` and `CLAUDE.md` remain consistent.
- Generated app code stays under `output/{project-name}/`.
