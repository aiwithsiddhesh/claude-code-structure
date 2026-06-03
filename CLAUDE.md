# AI Software Delivery Team Framework

A reusable multi-agent system for orchestrating software, AI, automation, and product development projects from idea to delivery. 10 specialized agents across 6 functional areas with defined roles, workflows, and quality gates.

**Core Principle**: Build the agent operating system first—documentation and frameworks before sample applications.

## Starting a Project

1. Fill in `project-intake.json` at the repo root — VS Code autocomplete works automatically via `project-intake.schema.json`
2. Run `/start-project` — reads the JSON, validates fields, maps stack to agents, sets output directory, produces the full delivery plan, initializes SPRINT.md with all backlog items set to `BA Status = needs-BA`
3. `business-analyst-agent` writes user stories and sets `BA Status = ready` for each item before sprint planning begins
4. Run `/sprint-plan` only after BA readiness is confirmed — items without `BA Status = ready` cannot be committed
5. All generated code goes to `output/{project.name}/` automatically

**Output Directory Rule**: all agents write project files to `output/{project-name}/`. Never write to the repo root. Defined in `coding-standards.md`.

## Workflow

The full delivery lifecycle with quality gates:

```
Intake → BA Readiness → Sprint Planning → Design/Ambiguity Gate → Task Start (DoR) → Development → Code Review → QA → Bug Verify/Regression → Sprint Review → Release → Retrospective
```

**What each gate catches:**

| Gate | Enforced By | Failure Class |
|---|---|---|
| Intake validation | `/start-project` | Missing/malformed fields |
| BA readiness | BA agent + `/sprint-plan` Step 3 | Vague features, no acceptance criteria before sprint commit |
| Sprint commit | `/sprint-plan` Steps 3–5 | Items without `ready` BA status or unresolved Medium/High ambiguity |
| Design/Ambiguity | `/design-doc` + SPRINT.md | Missing API/data/security decisions; unapproved architecture; ambiguity in any item regardless of size |
| Human approval | `/design-doc` Section 9 | Design self-approved by author with no independent judgment |
| DoR | `/task-start` Step 2 | Vague criteria, unresolved ambiguity, missing BA status reaching development |
| Code review | `/code-review` | Correctness, security, quality, test coverage |
| QA | `manual-functional-sdet` | Acceptance criteria not met, regressions |
| Bug verification | `/bug-verify` | Fix fails; status set to REJECTED (not IN FIX) — dev must explicitly resume |
| Sprint review | `/sprint-review` | Status drift, silent carry-forward |
| Release | `/release-checklist` | Reads SPRINT.md + TASK.md + BUG-N.md — unreleased bugs, missing sign-offs |

**When gates fail:**
- BA gate fails → item stays `needs-BA`; cannot enter sprint
- Design gate blocks → SPRINT.md shows `design-doc: blocked`; `/task-start` is blocked
- Design pending approval → SPRINT.md shows `design-doc: pending-approval`; dev cannot start
- DoR fails → TASK.md not created; agent returns to BA
- Bug REJECTED → BUG-N.md status = `REJECTED`; dev must resume and explicitly set to `IN FIX`
- Status conflict detected → skill stops and reports `STATUS CONFLICT`; do not proceed until resolved

## Architecture

1. **Agent-Centric** — Each agent has a defined role, inputs, and outputs. Communication happens through structured handoffs.
2. **Workflow-Driven** — Full lifecycle with quality gates at every stage. Gates are front-loaded — failures at planning cost far less than failures at QA or release.
3. **Template-Based** — Standardized artifacts reduce cognitive load and ensure consistency.
4. **Definition-Driven** — Clear DoR (definition of ready) and DoD (definition of done) prevent rework.
5. **Extensible** — New agent types and workflows can be added without breaking existing ones.

## Artifact Authority

When multiple documents exist, this precedence applies:
- **SPRINT.md** — authoritative for sprint membership, priority, assignment, release scope
- **TASK.md** — authoritative for task execution details and checkpoint history
- **BUG-N.md** — authoritative for bug lifecycle state and verification history
- **Design docs** — authoritative for architecture decisions, API contracts, approved tradeoffs

If sources conflict, the affected skill stops and reports `STATUS CONFLICT`. Never proceed from a known conflict.

## Development Approach

- **Document First** — Define agents in `.claude/agents/`, rules in `.claude/rules/`, skills in `.claude/skills/` before implementing behavior. Three rules are **path-scoped** and only load when editing matching files: `frontend-patterns.md` (React/TS/Tailwind — triggers on `.tsx/.ts/.jsx/.js`), `api-patterns.md` (REST conventions — triggers on routes/controllers/handlers), `database.md` (migrations/schema/Prisma — triggers on migration and model files). Always-on rules load every session: `team-workflow.md`, `agent-coordination.md`, `coding-standards.md`, `git-workflow.md`, `security.md`.
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
| `/agent-registry` | Full agent reference with assignment decision matrix and trigger conditions |
| `/start-project [description]` | Kick off a new project — validates intake, maps stack to agents, initializes SPRINT.md with all items as `BA Status = needs-BA`. Does not overwrite existing initialized SPRINT.md. |
| `/design-doc [feature name]` | Technical design document — required for Medium/Large/XL items AND any item with Ambiguity Medium/High; performs active ambiguity assessment (Section 0); requires human approval (Section 9) — not self-approval; syncs `design-doc` state back to SPRINT.md on every run |
| `/handoff [from] to [to]` | Structured handoff covering completed work, pending work, risks, and next actions |
| `/release-checklist [name]` | Pre-release quality gate — reads SPRINT.md + all TASK.md + all BUG-N.md; produces GO / NO-GO decision |
| `/status-report [project]` | Project status snapshot — reads SPRINT.md + all active TASK.md + all open BUG-N.md; detects and reports STATUS CONFLICTS |
| `/retrospective [project]` | Post-delivery retrospective with action items |
| `/sprint-plan [project]` | Commit backlog items to next sprint — enforces `BA Status = ready` gate; classifies Complexity AND Ambiguity; blocks items with unresolved Medium/High ambiguity; detects duplicate active sprint |
| `/sprint-review [project]` | Close current sprint, verify TASK.md state for each item, detect STATUS CONFLICTS, move incomplete items to backlog |
| `/change-request [project]` | Assess and decide a mid-sprint requirement change — detects duplicate CRs; records structured human decision with options considered, rationale, decider, date |
| `/bug-triage [project]` | Classify bug severity, detect duplicates, create BUG-{N}.md, assign to agent and sprint |
| `/bug-verify [project] [bug-id]` | Verify a bug fix — VERIFIED or REJECTED; on REJECTED sets status to REJECTED (not IN FIX); appends to Verification History (does not overwrite) |
| `/task-start [project] [task-id]` | Break sprint task into atomic steps, create TASK.md — enforces BA Status, Ambiguity, and design-doc DoR checks; records DoR verification result in TASK.md |
| `/task-checkpoint [project] [task-id]` | Write session checkpoint — handles IN PROGRESS, BLOCKED, and REWORK flows; guards against duplicate Completion entries |
| `/task-resume [project] [task-id]` | Resume task — checks state and handles BLOCKED, REWORK, and ON_HOLD correctly; detects STATUS CONFLICTS |
| `/code-review [project] [task-id]` | Structured code review before QA — APPROVED or CHANGES REQUESTED; hiring-manager-orchestrator may never be reviewer |

<!-- Maintainer note: Agent descriptions live in .claude/agents/. Do not duplicate them here.
     Always-on rules live in .claude/rules/:
       team-workflow.md       — handoff protocol, quality gate map, state machine, escalation
       agent-coordination.md  — assignment matrix, parallel/sequential rules
       coding-standards.md    — naming, formatting, output directory convention
       git-workflow.md        — branch naming, conventional commits, PR rules
       security.md            — non-negotiable security rules for all agents
       frontend-patterns.md   — path-scoped: React/TS/Tailwind conventions
       api-patterns.md        — path-scoped: REST naming, response format, versioning
       database.md            — path-scoped: migrations, schema, query patterns -->
