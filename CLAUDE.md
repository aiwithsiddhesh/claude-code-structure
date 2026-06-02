# AI Software Delivery Team Framework

A reusable multi-agent system for orchestrating software, AI, automation, and product development projects from idea to delivery. 10 specialized agents across 6 functional areas with defined roles, workflows, and quality gates.

**Core Principle**: Build the agent operating system first—documentation and frameworks before sample applications.

## Starting a Project

1. Fill in `project-intake.json` at the repo root — VS Code autocomplete works automatically via `project-intake.schema.json`
2. Run `/start-project` — reads the JSON, validates fields, maps stack to agents, sets output directory, produces the full delivery plan
3. All generated code goes to `output/{project.name}/` automatically

**Output Directory Rule**: all agents write project files to `output/{project-name}/`. Never write to the repo root. Defined in `coding-standards.md`.

## Architecture

1. **Agent-Centric** — Each agent has a defined role, inputs, and outputs. Communication happens through structured handoffs.
2. **Workflow-Driven** — Linear lifecycle (intake → planning → development → QA → release → retrospective) with quality gates at each stage.
3. **Template-Based** — Standardized artifacts reduce cognitive load and ensure consistency.
4. **Definition-Driven** — Clear DoR (definition of ready) and DoD (definition of done) prevent rework.
5. **Extensible** — New agent types and workflows can be added without breaking existing ones.

## Development Approach

- **Document First** — Define agents in `.claude/agents/`, rules in `.claude/rules/`, skills in `.claude/skills/` before implementing behavior.
- **Handoff Clarity** — Every stage states who inputs, who processes, who receives output. Use `/handoff` skill at every transition.
- **Quality Gates** — Measurable acceptance criteria at each workflow stage, defined in `.claude/rules/team-workflow.md`.
- **Example-Driven** — Run `/start-project` on a real scenario to validate the framework; if it doesn't produce a coherent plan, revise the intake or rules.

## Common Tasks

| Task | Action |
|---|---|
| Add a new agent role | Create `.claude/agents/{name}.md` with name, description, model, color, memory frontmatter + role body |
| Add a workflow rule | Add to `.claude/rules/team-workflow.md` (always-on) or create a new `.claude/rules/{topic}.md` |
| Add a path-scoped rule | Create `.claude/rules/{topic}.md` with `paths:` frontmatter listing glob patterns |
| Add a reusable workflow | Create `.claude/skills/{name}/SKILL.md` with description frontmatter |
| Document a process | Add to `.claude/rules/` if it applies every session, or `.claude/skills/` if on-demand |

## Documentation Standards

- Use Markdown for all documentation.
- Start every document with a 1–2 sentence purpose statement.
- Use structured sections (Inputs, Outputs, Responsibilities, Success Criteria).
- Provide examples wherever possible.
- Break large topics across multiple files.

## Available Skills

| Command | Purpose |
|---|---|
| `/agent-registry` | Full agent reference with assignment decision matrix |
| `/start-project [description]` | Kick off a new project with orchestrator assignment planning |
| `/design-doc [feature name]` | Technical design document — run before development begins |
| `/handoff [from] to [to]` | Generate structured handoff between agents |
| `/release-checklist [name]` | Pre-release quality gate verification |
| `/status-report [project]` | Project status summary across all active agents |
| `/retrospective [project]` | Post-delivery retrospective with action items |
| `/sprint-plan [project]` | Commit backlog items to next sprint, assign agents |
| `/sprint-review [project]` | Close current sprint, move incomplete items to backlog |
| `/change-request [project]` | Assess and decide a mid-sprint requirement change |
| `/bug-triage [project]` | Classify bug severity, create BUG-{N}.md, assign to agent and sprint |
| `/bug-verify [project] [bug-id]` | Verify a bug fix — VERIFIED or REJECTED with reason |
| `/task-start [project] [task-id]` | Break sprint task into atomic steps, create TASK.md |
| `/task-checkpoint [project] [task-id]` | Write session checkpoint before ending incomplete work |
| `/task-resume [project] [task-id]` | Resume incomplete task from last checkpoint in new session |

<!-- Maintainer note: Agent descriptions live in .claude/agents/. Do not duplicate them here.
     Always-on rules live in .claude/rules/:
       team-workflow.md       — handoff protocol, quality gates, escalation
       agent-coordination.md  — assignment matrix, parallel/sequential rules
       coding-standards.md    — naming, formatting, output directory convention
       git-workflow.md        — branch naming, conventional commits, PR rules
       security.md            — non-negotiable security rules for all agents
       frontend-patterns.md   — path-scoped: React/TS/Tailwind conventions
       api-patterns.md        — path-scoped: REST naming, response format, versioning
       database.md            — path-scoped: migrations, schema, query patterns -->
