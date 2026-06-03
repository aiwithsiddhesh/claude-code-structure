# AI Software Delivery Team Framework

A production-grade multi-agent system for Claude Code that orchestrates software delivery from idea to release. Ten specialized agents, sprint-based workflow, context-aware task management, and a complete bug lifecycle — all enforced through structured rules, skills, and living state documents.

---

## Table of Contents

1. [What This Is](#what-this-is)
2. [Prerequisites](#prerequisites)
3. [Directory Structure](#directory-structure)
4. [Framework Architecture](#framework-architecture)
5. [The Agent Team](#the-agent-team)
6. [Rules Reference](#rules-reference)
7. [Complete Project Workflow](#complete-project-workflow)
8. [Sprint Lifecycle](#sprint-lifecycle)
9. [Task Lifecycle](#task-lifecycle)
10. [Bug Lifecycle](#bug-lifecycle)
11. [Skills Reference](#skills-reference)
12. [Project State Files](#project-state-files)
13. [Context Loading Strategy](#context-loading-strategy)
14. [Extending the Framework](#extending-the-framework)

---

## What This Is

This framework turns Claude Code into a structured software delivery team. Instead of ad-hoc prompting, every project follows a defined process:

- A **project intake form** captures requirements, stack, timeline, and constraints before any work begins
- An **orchestrator agent** assigns the right specialists based on what the project actually needs
- **Sprint planning** breaks work into weekly cycles with committed scope
- **Task management** breaks sprint items into atomic steps that fit within a single context window — with checkpoints so no progress is lost between sessions
- **Bug tracking** has a full lifecycle from discovery through fix, verification, and regression test
- **Quality gates** prevent release without QA sign-off at every stage

Everything is coordinated through three living state documents per project: `SPRINT.md`, `TASK.md` files, and `BUG-{N}.md` files.

---

## Prerequisites

- [Claude Code](https://code.claude.com) installed and configured
- A project directory where you want to deliver software

No other setup required. Drop this framework into your project directory and start.

---

## Directory Structure

```
your-project/
├── README.md                          ← this file
├── CLAUDE.md                          ← project instructions (loaded every session)
├── CLAUDE.local.md                    ← personal preferences (gitignore this)
├── project-intake.json                ← fill before starting any project
├── project-intake.schema.json         ← VS Code autocomplete + validation
│
├── .vscode/
│   └── settings.json                  ← wires schema to intake file automatically
│
├── .claude/
│   ├── agents/                        ← 10 specialist agents
│   │   ├── hiring-manager-orchestrator.md
│   │   ├── business-analyst-agent.md
│   │   ├── frontend-engineer.md
│   │   ├── backend-engineer.md
│   │   ├── full-stack-engineer.md
│   │   ├── devops-cloud-engineer.md
│   │   ├── genai-llm-engineer.md
│   │   ├── data-ml-engineer.md
│   │   ├── manual-functional-sdet.md
│   │   └── automation-sdet-agent.md
│   │
│   ├── rules/                         ← always-on and path-scoped rules
│   │   ├── team-workflow.md           ← handoffs, quality gates, sprint rules, bug states
│   │   ├── agent-coordination.md      ← who to assign, parallel vs sequential
│   │   ├── coding-standards.md        ← naming, formatting, task lifecycle protocol
│   │   ├── git-workflow.md            ← branches, conventional commits, PR rules
│   │   ├── security.md                ← non-negotiables across all agents
│   │   ├── frontend-patterns.md       ← path-scoped: React/TS/Tailwind conventions
│   │   ├── api-patterns.md            ← path-scoped: REST naming, response envelope
│   │   └── database.md                ← path-scoped: migrations, schema, Prisma
│   │
│   └── skills/                        ← on-demand workflows, invoked with /command
│       ├── agent-registry/            ← /agent-registry
│       ├── start-project/             ← /start-project
│       ├── design-doc/                ← /design-doc
│       ├── handoff/                   ← /handoff
│       ├── sprint-plan/               ← /sprint-plan
│       ├── sprint-review/             ← /sprint-review
│       ├── change-request/            ← /change-request
│       ├── bug-triage/                ← /bug-triage
│       ├── bug-verify/                ← /bug-verify
│       ├── task-start/                ← /task-start
│       ├── task-checkpoint/           ← /task-checkpoint
│       ├── task-resume/               ← /task-resume
│       ├── code-review/               ← /code-review
│       ├── release-checklist/         ← /release-checklist
│       ├── status-report/             ← /status-report
│       └── retrospective/             ← /retrospective
│
├── assets/                            ← templates used by /design-doc, /start-project, /bug-triage
│   ├── design-doc-template.md         ← filled by /design-doc for each feature
│   ├── assignment-plan.md             ← filled by /start-project at project kickoff
│   ├── sprint-md-template.md          ← used to initialize output/{project}/SPRINT.md
│   └── bug-template.md                ← filled by /bug-triage for each BUG-{N}.md
│
└── output/                            ← all generated project code lives here
    └── {project-name}/
        ├── SPRINT.md                  ← sprint state (single source of truth)
        ├── .tasks/
        │   └── S1-1.md                ← one TASK.md per sprint task
        ├── .bugs/
        │   └── BUG-001.md             ← one BUG file per bug
        └── src/                       ← generated application code
```

---

## Framework Architecture

### Three-Layer System

The framework manages three distinct boundaries:

```
┌─────────────────────────────────────────┐
│  PROJECT (project-intake.json)           │  What to build, who builds it,
│  Scope, stack, timeline, constraints     │  deadline, output directory
├─────────────────────────────────────────┤
│  SPRINT (SPRINT.md)                      │  Which tasks are committed
│  Weekly cycles, backlog, change log      │  this week, what's pending
├─────────────────────────────────────────┤
│  TASK / CONTEXT (.tasks/, .bugs/)        │  Atomic steps that fit in one
│  Checkpoints, file lists, resume points  │  Claude Code context window
└─────────────────────────────────────────┘
```

Each layer has its own state document. When sources conflict, defined precedence applies: SPRINT.md is authoritative for sprint membership and scope; TASK.md is authoritative for task execution state; BUG-{N}.md is authoritative for bug lifecycle state. Skills stop and report `STATUS CONFLICT` rather than silently picking one source.

### Context Loading Architecture

Not all files are loaded in every session. The framework is designed to minimize unnecessary context:

| Layer | Files | When Loaded |
|---|---|---|
| **Always** | `CLAUDE.md` + 5 unconditional rules | Every session |
| **Path-scoped** | `frontend-patterns.md`, `api-patterns.md`, `database.md` | Only when touching matching file paths |
| **Per-agent** | Agent file (avg 80 lines) | Only when that agent is invoked |
| **On-demand** | 15 skills | Only when you run `/command` |

Path-scoped rules include both standard paths (`src/**`) and output paths (`output/**/src/**`) so they trigger correctly regardless of where generated code lives.

---

## The Agent Team

### Orchestration

| Agent | Model | Role |
|---|---|---|
| `hiring-manager-orchestrator` | Opus | Coordinates the whole team. Reads SPRINT.md and `.bugs/` before every decision. Never writes code. |

### Requirements

| Agent | Model | Role |
|---|---|---|
| `business-analyst-agent` | Sonnet | Requirements, user stories, acceptance criteria, DoR checklist, story sizing |

### Development

| Agent | Model | Role |
|---|---|---|
| `frontend-engineer` | Sonnet | React, Next.js, TypeScript, Tailwind, accessibility, API integration |
| `backend-engineer` | Sonnet | REST APIs, databases, auth/authz, business logic, migrations |
| `full-stack-engineer` | Sonnet | End-to-end features when UI + API + DB work is tightly coupled |
| `devops-cloud-engineer` | Sonnet | CI/CD, Docker, cloud infra, monitoring, deployment |
| `genai-llm-engineer` | Sonnet | RAG pipelines, prompt engineering, LLM integrations, embeddings |
| `data-ml-engineer` | Sonnet | Data pipelines, feature engineering, ML model training and evaluation |

### Quality Assurance

| Agent | Model | Role |
|---|---|---|
| `manual-functional-sdet` | Sonnet | Manual testing, bug documentation, fix verification, QA sign-off. Sets `REWORK` on rejected tasks with exact defects. Marks qualifying test cases `AUTOMATION READY` for self-handoff to automation agent. |
| `automation-sdet-agent` | Sonnet | Automated test suites, CI integration, regression tests for verified bugs. Self-triggers from `AUTOMATION READY` tags in manual QA notes — no orchestrator assignment needed. |

### Assignment Decision Matrix

| You need | Assign |
|---|---|
| Multi-agent coordination, blockers, release | `hiring-manager-orchestrator` |
| Requirements clarification, user stories | `business-analyst-agent` |
| UI only | `frontend-engineer` |
| API / DB / backend only | `backend-engineer` |
| Full feature (UI + API + DB, tightly coupled) | `full-stack-engineer` |
| Infra, CI/CD, deployment | `devops-cloud-engineer` |
| LLM, chatbot, RAG, embeddings | `genai-llm-engineer` |
| Data pipelines, ML models | `data-ml-engineer` |
| Manual QA, feature sign-off | `manual-functional-sdet` |
| Automated tests, CI suite | `automation-sdet-agent` |

**Rules:**
- `hiring-manager-orchestrator` never writes code
- `genai-llm-engineer` and `data-ml-engineer` are only assigned when the project genuinely needs AI/ML
- `manual-functional-sdet` is mandatory before every release — cannot be skipped via `skip_agents`

---

## Rules Reference

### Always-On Rules (loaded every session)

**`team-workflow.md`** — The master process document. Covers the full project lifecycle, handoff protocol (7 required fields), quality gates at each stage, escalation paths, sprint cycle, change control decisions, bug severity sprint rules, task checkpoint quality gate, and the bug lifecycle state machine.

**`agent-coordination.md`** — Assignment matrix, self-limitation rules, parallel vs sequential work guidance, QA timing rules.

**`coding-standards.md`** — Naming conventions for TypeScript/JavaScript/Python/database, strict TypeScript rules, file and function length limits, comment standards, formatter config, the code review process (what "code-reviewed" means, automated checks, reviewer ownership), and the mandatory task lifecycle protocol (task-start → checkpoint → code-review → checkpoint completion → QA).

**`git-workflow.md`** — Branch naming (`feature/`, `fix/`, `chore/`, `docs/`, `refactor/`), Conventional Commits format with all types, PR description template, PR size limits (400 target / 800 hard), review rules, merge strategy, semver tagging.

**`security.md`** — Non-negotiable rules: secrets management, input validation, parameterized queries, auth patterns, HTTP security headers, dependency scanning, logging restrictions, file operation safety. Includes a code-review security checklist.

### Path-Scoped Rules (loaded only when touching matching files)

**`frontend-patterns.md`** — Triggers on `src/**/*.{ts,tsx,jsx,js}`, `pages/**`, `app/**`, `components/**`, and all `output/**` equivalents. Covers: component anatomy, folder structure, state management hierarchy (local → React Query → Zustand), Tailwind usage, react-hook-form + Zod pattern, accessibility baseline.

**`api-patterns.md`** — Triggers on `src/api/**`, `**/routes/**`, `**/controllers/**`, `**/handlers/**`, and all `output/**` equivalents. Covers: URL conventions, standard response envelope `{ data, meta }`, error envelope `{ error: { code, message, details } }`, HTTP status codes, pagination, versioning, per-route auth declaration.

**`database.md`** — Triggers on `**/migrations/**`, `**/models/**`, `**/schemas/**`, `prisma/**`, and all `output/**` equivalents. Covers: migration naming and rollback requirements, schema conventions, index naming, N+1 prevention, transaction boundaries, Prisma patterns.

---

## Complete Project Workflow

### Step 1 — Fill the Intake Form

Open `project-intake.json`. VS Code will show autocomplete and validation automatically (wired via `.vscode/settings.json`).

```json
{
  "project": {
    "name": "customer-portal",
    "display_name": "Customer Portal",
    "description": "Self-service portal for enterprise customers to manage subscriptions and invoices.",
    "type": "web-app"
  },
  "stack": {
    "frontend": "next.js",
    "backend": "node.js-express",
    "database": "postgresql",
    "auth": "jwt",
    "hosting": "aws",
    "ai_ml": false
  },
  "requirements": {
    "features": [
      "User registration and login with email/password",
      "Dashboard showing subscription status and usage metrics",
      "Invoice list with PDF download",
      "Role-based access: admin sees all users, user sees own data"
    ],
    "out_of_scope": ["Payment processing", "Mobile app"],
    "non_functional": {
      "expected_users": "1k-10k",
      "performance": "API p95 < 200ms",
      "security_level": "high"
    }
  },
  "timeline": {
    "deadline": "2026-08-01",
    "milestones": [
      { "name": "MVP — auth + dashboard", "date": "2026-07-01" }
    ]
  }
}
```

**Intake field mapping to agents:**

| Field | Effect |
|---|---|
| `project.name` | Becomes `output/{name}/` — all code goes here |
| `project.type` | Sets default agent roster |
| `stack.frontend` | Assigns `frontend-engineer`, loads `frontend-patterns.md` |
| `stack.backend` | Assigns `backend-engineer`, loads `api-patterns.md` + `database.md` |
| `stack.hosting` | Assigns `devops-cloud-engineer` |
| `stack.ai_ml: true` | Assigns `genai-llm-engineer` or `data-ml-engineer` based on `ai_ml_type` |
| `requirements.features` | `business-analyst-agent` converts to user stories |
| `requirements.out_of_scope` | Hard boundary — no agent builds these |
| `security_level: high` | Mandatory security review on every PR, pen test before release |
| `timeline.deadline` | Orchestrator flags scope vs timeline as FEASIBLE or AT RISK |
| `team.skip_agents` | Removes agents from auto-assignment (cannot remove `manual-functional-sdet`) |
| `context.constraints` | Hard rules passed to every agent |
| `context.existing_codebase: true` | All agents read existing code before writing |

### Step 2 — Start the Project

```
/start-project
```

The skill reads `project-intake.json`, validates all required fields, auto-sets `output.dir` to `output/{project.name}/`, maps stack to agents, and produces:
- Full Agent Assignment Plan with tasks per agent and a full Quality Gate Map
- Delivery Sequence (Phase 1: BA Readiness → Phase 2: Design → Phase 3: Development → Phase 4: QA/Release)
- Quality gates per phase
- Timeline feasibility assessment
- `output/{project.name}/SPRINT.md` initialized with all features in the backlog, each with `BA Status = needs-BA`

> **Re-run safe:** `/start-project` will not overwrite an existing initialized SPRINT.md. It stops and asks you to confirm before proceeding.

### Step 3 — Requirements and Design

**BA readiness must complete before sprint planning.** `business-analyst-agent` converts intake features into user stories, assigns each a priority, size estimate, and DoR status, and updates `BA Status` in SPRINT.md to `ready` for each item:

```
As a [user type]
I want to [action]
So that [benefit]

Acceptance Criteria:
- Given [context], when [action], then [outcome]
```

Once items are `ready`, run design docs before committing to a sprint:

```
/design-doc [feature-name]
```

Required for: **Medium, Large, or XL** features, and **any item with Ambiguity = Medium or High** regardless of size (Small items are not exempt when ambiguity is unresolved).

`/design-doc` performs an **active ambiguity assessment** (Section 0) — it explicitly identifies missing API decisions, missing data/schema decisions, unresolved tradeoffs, and stakeholder questions. If any are open, it emits `⚠️ DESIGN BLOCKED`. If ambiguity is resolved but no human approval exists yet, it emits `⚠️ DESIGN PENDING APPROVAL` — a named non-author approver must complete Section 9 (Approval Record) with options considered, chosen option, rationale, and date before the design is approved.

**Every run of `/design-doc` automatically syncs its state back to SPRINT.md**: `design-doc: done`, `design-doc: pending-approval`, or `design-doc: blocked`. `/task-start` reads this column and blocks development until the state is `done` or `waived`.

### Step 4 — Sprint Planning

```
/sprint-plan customer-portal
```

The orchestrator reads `SPRINT.md` and:
1. **BA Readiness Check** — items with `BA Status = needs-BA` or `blocked` cannot be committed. Only `ready` items proceed.
2. **Complexity + Ambiguity classification** — each item gets both ratings.
3. **Design and Ambiguity Gate** — Medium/Large/XL items require `design-doc: done/waived`; any item with Ambiguity Medium/High requires it too regardless of size.
4. Commits realistic scope, assigns agents, writes plan to SPRINT.md.

### Step 5 — Development (Task Lifecycle)

Each agent follows the three-command task protocol:

```
/task-start customer-portal S1-2
```
Breaks the sprint task into atomic steps, each producing one verifiable artifact. Creates `.tasks/S1-2.md`. No code is written before this exists.

```
# Agent works through steps, checking each off in TASK.md
# Session ending before task is complete:
/task-checkpoint customer-portal S1-2
```
Records completed steps, all files written, current state, and exact resume instructions. Updates SPRINT.md progress.

```
# New session continuing the same task:
/task-resume customer-portal S1-2
```
Reads the last checkpoint. Agent continues from the exact stopping point. Does not re-read the whole codebase.

### Step 6 — Mid-Sprint Events

**Requirement change:**
```
/change-request customer-portal
```
Assesses change size (Trivial/Small/Medium/Large/Breaking), impact on current sprint, and produces a formal decision: **Absorb** (into current sprint), **Defer** (to backlog), or **Reject** (with reason). Records in `SPRINT.md` change log.

**Bug discovered:**
```
/bug-triage customer-portal
```
Classifies severity, assigns owner agent, creates `output/customer-portal/.bugs/BUG-{N}.md` with full reproduction steps and acceptance criteria for the fix. Critical bugs pause lower-priority work immediately.

### Step 7 — QA and Bug Resolution

When a task is complete, the dev agent runs `/code-review` first. Once APPROVED, they run `/task-checkpoint` completion flow to set the task to `READY FOR QA`.

`manual-functional-sdet` reads `.tasks/{task-id}.md` and verifies:
1. All steps are checked off
2. Code review APPROVED is recorded in the Completion entry

If either check fails, the task is returned to `REWORK` status with exact defects documented.

When a dev agent completes a bug fix:
1. Updates `BUG-{N}.md` Fix section with files changed and description
2. Sets status to `READY FOR VERIFICATION`

```
/bug-verify customer-portal BUG-001
```
`manual-functional-sdet` re-runs reproduction steps and validates acceptance criteria. Result is **VERIFIED** or **REJECTED** with exact reason and evidence appended to the Verification History (each attempt is a permanent record, never overwritten).

On **REJECTED**: status is set to `REJECTED` — not `IN FIX`. The dev agent must explicitly resume work (`/task-resume` or `/task-start`) and set status to `IN FIX` when actively working again. This prevents silent re-entry without acknowledging the rejection reason.

On **VERIFIED**: `automation-sdet-agent` picks up the test case and writes a regression test, marking the bug **CLOSED**.

`automation-sdet-agent` also self-triggers from manual QA notes: test cases marked `AUTOMATION READY` are picked up without waiting for orchestrator assignment.

### Step 8 — Sprint Review

```
/sprint-review customer-portal
```
Reviews each committed item (DONE / PARTIAL / NOT STARTED). Verifies TASK.md completion for every item. Moves incomplete items back to backlog with notes. Writes sprint to history. Triggers follow-up recommendations.

### Step 9 — Release

When all sprint features are complete and QA has signed off:
```
/release-checklist customer-portal
```
Runs through all gates: development completeness, testing coverage, documentation, infrastructure, security, and release readiness. Produces a GO / NO-GO / CONDITIONAL GO decision.

### Step 10 — Retrospective

```
/retrospective customer-portal
```
Covers: delivery summary vs planned scope, timeline variance, What Went Well / What Could Be Better / Surprises tables, action items (with owner + date + success metric), framework improvement recommendations, agent performance notes, 6 delivery metrics, and sign-off.

---

## Sprint Lifecycle

```
/start-project
     │ (initializes SPRINT.md — all items BA Status = needs-BA)
     ▼
business-analyst-agent writes user stories
  → sets BA Status = ready for each item in SPRINT.md
     │
     ▼
/design-doc [feature]  (for M/L/XL items and Medium/High ambiguity items)
  → syncs design-doc: done/pending-approval/blocked to SPRINT.md
     │
     ▼
/sprint-plan ──────────────────────────────────────────┐
  (only ready items + resolved ambiguity can be         │ (repeat each sprint)
   committed)                                           │
     │                                                   │
     ▼                                                   │
[Agents work tasks]                                      │
  /task-start {project} {task-id}                        │
  /task-checkpoint {project} {task-id}  (end of session) │
  /task-resume {project} {task-id}      (new session)    │
     │                                                   │
     ├──── Requirement changes? → /change-request ───────┤ (Absorb/Defer/Reject)
     ├──── Bug found?           → /bug-triage      ───── see Bug Lifecycle
     │                                                   │
     ▼                                                   │
/sprint-review ─────────────────────────────────────────┘
  (reads TASK.md for each item, detects STATUS CONFLICTS)
     │
     ▼ (backlog empty, all QA signed off)
/release-checklist
  (reads SPRINT.md + all TASK.md + all BUG-N.md)
     │
     ▼
/retrospective
```

**Sprint rules:**
- New work never enters an active sprint without a `/change-request` decision
- `SPRINT.md` is authoritative for sprint membership and scope; `TASK.md` is authoritative for task execution state; `BUG-N.md` is authoritative for bug state — conflicts between sources trigger `STATUS CONFLICT`, not silent resolution
- Incomplete items return to backlog at review — never auto-carry silently
- BLOCKED and ON_HOLD tasks require an explicit orchestrator decision at sprint review — never silently carried
- `manual-functional-sdet` removal from `skip_agents` is blocked — QA is mandatory

**Change control authority:**
- Trivial/Small changes: orchestrator decides alone, same session
- Medium changes: orchestrator + product owner must agree; auto-defer if PO unavailable
- Large/Breaking changes: orchestrator + product owner required; sprint may pause for replanning
- Affected tasks move to `ON_HOLD` during Medium/Large change review; orchestrator releases the hold after decision

---

## Task Lifecycle

Every sprint task and bug fix follows the same protocol:

```
┌─────────────────────────────────────────────────────────────┐
│  /task-start {project} {task-id}                             │
│  → Reads sprint task details                                 │
│  → Forces atomic step decomposition (max 10 steps)          │
│  → Creates output/{project}/.tasks/{task-id}.md              │
│  → Status: IN PROGRESS                                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
              [Agent works through steps]
              [Checks off each step in TASK.md]
                       │
         ┌─────────────┼──────────────┐
         │             │              │
    Hit blocker?  Session ending? All steps done?
         │             │              │
         ▼             ▼              ▼
  /task-checkpoint  /task-checkpoint  /code-review
  [BLOCKED flow]    [Standard flow]   → APPROVED?
  Status: BLOCKED        │                │
                    [New session]    /task-checkpoint
                         │          [Completion flow]
                         ▼          Status: READY FOR QA
                  /task-resume            │
                  → Checks state          ▼
                  → BLOCKED?      manual-functional-sdet
                    (verify        validates TASK.md +
                     blocker       code review record
                     resolved)          │
                  → REWORK?        ┌────┴────┐
                    (read QA       │         │
                     defects)    DONE      REWORK
                  → ON_HOLD?    (sign-off) (set by QA
                    (STOP —                 with defects)
                     contact
                     orchestrator)
```

**Task state machine:**

| State | Who Sets It | Meaning |
|---|---|---|
| `IN PROGRESS` | Dev agent | Actively being worked |
| `BLOCKED` | Dev agent | Waiting on an external dependency — blocker and owner documented |
| `REWORK` | `manual-functional-sdet` | QA rejected — specific defects must be fixed |
| `ON_HOLD` | Orchestrator only | Deliberately paused — only orchestrator releases this |
| `READY FOR QA` | Dev agent (completion flow) | All steps done, code review APPROVED |
| `DONE` | `manual-functional-sdet` | QA signed off |

**Atomic step rule:** A step must produce one verifiable artifact — a file, a passing test, a working endpoint.

**Task is DONE only when:** all steps checked off + code review APPROVED + `manual-functional-sdet` sign-off. An agent self-reporting done is not done.

---

## Bug Lifecycle

```
Anyone finds a bug
       │
       ▼
/bug-triage {project}
  Creates output/{project}/.bugs/BUG-{N}.md with:
  - Exact reproduction steps
  - Expected vs actual behaviour
  - Severity classification
  - Acceptance criteria for the fix
  - Owner agent assigned
  Updates SPRINT.md with one-line reference
       │
       ▼
Dev agent reads BUG-{N}.md acceptance criteria
  /task-start {project} BUG-{N}   ← same lifecycle as sprint tasks
  [fixes the bug]
  Updates BUG-{N}.md Fix section (files changed + description)
  Sets status → READY FOR VERIFICATION
       │
       ▼
/bug-verify {project} BUG-{N}
  manual-functional-sdet re-runs exact reproduction steps
  Validates all acceptance criteria
       │
       ├── REJECTED → Verification History entry appended with exact reason,
       │              evidence, affected criteria, resubmission instructions
       │              status → REJECTED (NOT IN FIX)
       │              Dev agent must /task-resume or /task-start, then
       │              explicitly set status → IN FIX when actively working
       │
       └── VERIFIED → automation-sdet-agent triggered
                      Writes regression test from reproduction steps
                      Confirms CI passing
                      Sets status → CLOSED
```

**Bug states in BUG-{N}.md:**

| State | Set By | Meaning |
|---|---|---|
| `OPEN` | `/bug-triage` | Documented, assigned, not yet in fix |
| `IN FIX` | Dev agent | Actively being fixed |
| `READY FOR VERIFICATION` | Dev agent | Fix complete, awaiting QA |
| `REJECTED` | `/bug-verify` | Verification failed — dev must explicitly resume and set to IN FIX when working |
| `VERIFIED` | `/bug-verify` | Fix confirmed, awaiting regression test |
| `CLOSED` | `automation-sdet-agent` | Regression test written and passing in CI |

**A bug is not done until CLOSED.** SPRINT.md holds one-line references; `BUG-{N}.md` holds the full state. Always read the bug file for accurate status.

---

## Skills Reference

| Command | Agent | Purpose |
|---|---|---|
| `/agent-registry` | Any | Full agent reference with capabilities, triggers, and assignment decision matrix |
| `/start-project` | Orchestrator | Validate intake, map stack to agents, produce delivery plan, initialize SPRINT.md with all items as `BA Status = needs-BA`. Will not overwrite an existing initialized SPRINT.md. |
| `/design-doc [feature]` | Lead engineer | Required for M/L/XL items and any item with Ambiguity Medium/High. Performs active ambiguity assessment (Section 0). Requires human approval in Section 9 — author cannot self-approve. Syncs `design-doc` state back to SPRINT.md on every run. |
| `/handoff [from] to [to]` | Any | Structured handoff covering completed work, pending work, acceptance criteria, and next actions |
| `/sprint-plan [project]` | Orchestrator | Enforces BA readiness gate (no `needs-BA` items committed); classifies Complexity and Ambiguity; blocks unresolved Medium/High ambiguity; detects duplicate active sprint |
| `/sprint-review [project]` | Orchestrator | Close sprint — reads TASK.md for every item, detects STATUS CONFLICTS, explicitly decides BLOCKED/ON_HOLD items |
| `/change-request [project]` | Orchestrator | Assess mid-sprint requirement change — detects duplicate CRs; records structured decision with options/rationale/decider/date in SPRINT.md Human Decisions |
| `/bug-triage [project]` | Orchestrator + SDET | Check for duplicates first; classify bug; create BUG-{N}.md; assign |
| `/bug-verify [project] [bug-id]` | manual-functional-sdet | Verify fix — VERIFIED or REJECTED; on REJECTED sets status to REJECTED (not IN FIX) and appends to Verification History |
| `/task-start [project] [task-id]` | Dev agent | Enforces BA Status, Ambiguity, and design-doc DoR checks before creating TASK.md; records DoR verification result in TASK.md |
| `/task-checkpoint [project] [task-id]` | Dev agent | Write session checkpoint — standard, BLOCKED, or REWORK flows; guards against duplicate Completion entries |
| `/task-resume [project] [task-id]` | Dev agent | Resume task — checks state (IN PROGRESS / BLOCKED / REWORK / ON_HOLD); detects STATUS CONFLICTS |
| `/code-review [project] [task-id]` | Dev agent | Structured code review — APPROVED or CHANGES REQUESTED; hiring-manager-orchestrator may never be reviewer |
| `/status-report [project]` | Orchestrator | Reads SPRINT.md + all TASK.md + all BUG-N.md; detects STATUS CONFLICTS; shows dual-source status per task and bug |
| `/release-checklist [project]` | Orchestrator | Reads SPRINT.md + all TASK.md + all BUG-N.md; requires BUG-N.md CLOSED (not just VERIFIED) for release — GO / NO-GO / CONDITIONAL GO |
| `/retrospective [project]` | Orchestrator | Post-delivery review with action items |

---

## Project State Files

Three living documents track project state. Together they answer "where are we right now?" without reading through old design docs or handoffs.

### `output/{project}/SPRINT.md`

The authoritative source for sprint membership, priority, assignment, and release scope. All agents read this before making decisions. When SPRINT.md and TASK.md/BUG-N.md disagree on status, skills stop and report `STATUS CONFLICT`.

**Sections:**
- Project metadata (start date, deadline, sprint length, current sprint number)
- **Readiness Issues** — BA readiness failures and unresolved open questions blocking sprint commitment
- **Design Decisions** — design doc approvals, waivers, and blocked states; updated by `/design-doc`
- **Human Decisions** — every human decision recorded with options considered, chosen option, rationale, decider, date
- **State Conflicts** — status mismatches between SPRINT.md and TASK.md/BUG-N.md; cleared when resolved
- Current Sprint (committed items with Complexity, Ambiguity, Agent, Design Doc, Acceptance Criteria, Status columns)
- Backlog (all uncommitted work with BA Status, Ambiguity, and Acceptance Criteria columns)
- Change Requests (CR-N log with structured decision record)
- Bug Log (one-line reference per bug, links to BUG-{N}.md)
- Sprint History (completed sprints with metrics)

### `output/{project}/.tasks/{task-id}.md`

One file per sprint task. Created by `/task-start`, updated by `/task-checkpoint`.

**Sections:**
- Task identity (ID, agent, sprint, status)
- Acceptance criteria (copied from sprint commitment)
- Atomic steps with checkboxes
- Context risk flags
- Files this task will write
- Checkpoints (appended per session with completed steps, file list, state, resume point)
- Completion entry (when all steps done)

### `output/{project}/.bugs/BUG-{N}.md`

One file per bug. Created by `/bug-triage`, updated through the lifecycle. Authoritative for bug state — always read this file, not the SPRINT.md Bug Log summary.

**Sections:**
- Bug identity (ID, severity, type, status, dates, assignments) — status comment shows full state machine
- Reproduction steps + frequency + environment
- Expected vs actual behaviour
- Root cause hypothesis
- Acceptance criteria for fix
- Fix section (files changed, description — filled by dev agent)
- **Verification History** (append-only; each `/bug-verify` attempt adds a new entry with result, reason, evidence, affected criteria, resubmission instructions)
- Regression test section (test file, CI status — filled by automation-sdet-agent)

---

## Context Loading Strategy

The framework is designed to keep context lean. Here is what loads when:

**Every session (always-on rules):**
- `CLAUDE.md` — 77 lines
- `team-workflow.md` — lifecycle, gates, sprint rules, bug states
- `agent-coordination.md` — assignment matrix
- `coding-standards.md` — naming, task lifecycle protocol
- `git-workflow.md` — branch and commit conventions
- `security.md` — security non-negotiables

**Only when agent is invoked:**
- That agent's file (avg 80 lines)
- Agent reads `SPRINT.md` and relevant TASK.md / BUG files at runtime

**Only when touching matching files:**
- `frontend-patterns.md` — when editing `.tsx`, `.ts`, `.jsx`, `.js` files (including under `output/`)
- `api-patterns.md` — when editing routes, controllers, handlers (including under `output/`)
- `database.md` — when editing migrations, models, schemas, Prisma files (including under `output/`)

**Only when you run the command:**
- Any skill — loaded on demand, not in background context

---

## Extending the Framework

### Add a New Agent

Create `.claude/agents/{name}.md`:

```markdown
---
name: "your-agent-name"
description: "When to use this agent. Include examples with <example> tags."
model: sonnet
color: blue
memory: project
---

You are the [Role] for an AI Software Delivery Team. [Description]

## Core Responsibilities
...

## Bug Fix Protocol
[Copy from any existing coding agent]

## Task Lifecycle
[Copy from any existing coding agent]

## Definition of Done
...
```

Then add to `.claude/rules/agent-coordination.md` assignment matrix.

### Add a New Always-On Rule

Create `.claude/rules/{topic}.md` — no frontmatter needed for unconditional rules.

### Add a New Path-Scoped Rule

Create `.claude/rules/{topic}.md` with paths frontmatter:

```markdown
---
paths:
  - "src/{your-pattern}/**"
  - "output/**/{your-pattern}/**"
---
```

Always include both the standard path and the `output/**` equivalent so rules trigger correctly for generated project code.

### Add a New Skill

Create `.claude/skills/{name}/SKILL.md`:

```markdown
---
description: When this skill should be used. This text appears in the skills registry.
disable-model-invocation: true
argument-hint: "[argument description]"
---

# Skill Name — $ARGUMENTS

## Read State (if needed)
```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null`
```

## Step 1 — ...
```

Use `disable-model-invocation: true` for skills with side effects (they should only run when explicitly invoked).

Then add to the skills table in `CLAUDE.md` and the Available Skills table in this README.

---

## Quick Reference

### Starting a project
```
1. Fill project-intake.json
2. /start-project                          ← all items initialized as BA Status = needs-BA
3. [business-analyst-agent writes stories] ← sets BA Status = ready for each item
4. /design-doc [feature]                   ← for M/L/XL and Medium/High ambiguity items
5. /sprint-plan {project}                  ← only ready items with resolved ambiguity commit
6. [For each task] /task-start {project} {task-id}
```

### Completing a task (all steps done)
```
/code-review {project} {task-id}           ← must be APPROVED first
/task-checkpoint {project} {task-id}       ← writes Completion entry, sets READY FOR QA
```

### Ending a session mid-task
```
/task-checkpoint {project} {task-id}       ← standard, BLOCKED, or REWORK flow
```

### Starting a new session on existing task
```
/task-resume {project} {task-id}           ← checks state: IN PROGRESS / BLOCKED / REWORK / ON_HOLD
```

### Something changed mid-sprint
```
/change-request {project}
```

### Bug found
```
/bug-triage {project}
```

### Verifying a bug fix
```
/bug-verify {project} BUG-{N}
```

### End of sprint
```
/sprint-review {project}
/sprint-plan {project}    ← start next sprint
```

### Ready to release
```
/release-checklist {project}
/retrospective {project}
```
