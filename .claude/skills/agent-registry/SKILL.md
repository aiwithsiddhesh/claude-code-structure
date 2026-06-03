---
name: agent-registry
description: Full reference of all available agents, capabilities, and assignment guidance. Use when deciding which agent to assign to a task or when understanding the complete team composition. Reads the agent registry inline — no state files required. Outputs agent descriptions, triggers, stack, and the assignment decision matrix.
---

# Agent Registry

Complete reference for all 10 agents in the AI Software Delivery Team.

---

## 1. hiring-manager-orchestrator
**Role**: Team Orchestrator & Project Manager
**Model**: Opus (complex reasoning required)
**Use For**: Project kickoff, blocker resolution, progress tracking, release coordination, delivery summaries
**NOT For**: Direct development, coding, testing, deep technical work

**Assignment Triggers**:
- New project requires multiple agents
- Blockers need cross-agent coordination
- Status update across the team is needed
- Release gate validation required

---

## 2. business-analyst-agent
**Role**: Requirements & Business Analysis
**Use For**: Requirements gathering, user story writing, acceptance criteria, MVP scope, risk identification, ambiguity classification, SPRINT.md BA Status updates
**NOT For**: Implementation, coding, testing

**Assignment Triggers**:
- Project initiation — must run before `/sprint-plan`; sets BA Status = `ready` in SPRINT.md for each item
- Requirements unclear, ambiguous, or conflicting
- User stories or acceptance criteria needed
- Business logic needs documentation
- Sprint planning is blocked because items show `BA Status = needs-BA`

**Key output**: For every completed item, updates SPRINT.md backlog row with BA Status = `ready`, Acceptance Criteria, Complexity, Ambiguity (Low/Medium/High), and Priority. `/sprint-plan` will not commit items without this.

---

## 3. frontend-engineer
**Role**: UI/Frontend Development
**Stack**: React, Next.js, TypeScript, Tailwind CSS
**Use For**: UI screens, components, forms, API data display, frontend bugs, accessibility, responsive design
**NOT For**: Backend APIs, database, deployment, ML

---

## 4. backend-engineer
**Role**: API & Database Development
**Stack**: Node.js, Python, PostgreSQL, REST, GraphQL
**Use For**: API design/implementation, database modeling, business logic, auth/authz, backend optimization
**NOT For**: Frontend, deployment/infra, ML

---

## 5. full-stack-engineer
**Role**: End-to-End Feature Development
**Use For**: Features spanning UI + API + DB when work can't be cleanly parallelized; cross-stack debugging
**NOT For**: Infra, ML, requirements

**Use Instead of frontend + backend when**:
- Feature is tightly coupled (form → API → DB → UI response)
- Cross-stack debugging is needed
- Team is working sequentially, not in parallel

---

## 6. devops-cloud-engineer
**Role**: Infrastructure & DevOps
**Stack**: Docker, GitHub Actions, AWS/GCP/Azure, Terraform
**Use For**: CI/CD pipelines, containerization, cloud infra, monitoring, deployment automation, environment setup
**NOT For**: Application code, ML, frontend/backend business logic

---

## 7. genai-llm-engineer
**Role**: AI/LLM Feature Development
**Stack**: OpenAI/Anthropic APIs, LangChain, vector databases, embeddings
**Use For**: Chatbots, RAG pipelines, prompt engineering, LLM integrations, AI agent workflows
**NOT For**: General engineering, standard APIs, frontend/backend without AI

⚠️ Only assign when project explicitly involves LLM/AI features.

---

## 8. data-ml-engineer
**Role**: Data & ML Engineering
**Stack**: Python, Pandas, scikit-learn, MLflow, SQL
**Use For**: Data pipelines, feature engineering, ML model training/evaluation, predictions, data quality
**NOT For**: LLM/RAG (use genai-llm-engineer), general backend, frontend

⚠️ Only assign when project explicitly involves ML models or data pipelines.

---

## 9. manual-functional-sdet
**Role**: Manual QA & Functional Testing
**Use For**: Pre-release validation, feature acceptance testing, bug verification, regression testing, QA sign-off
**NOT For**: Automated test suites, application code

**Timing**: Engage AFTER development is complete (not before).

---

## 10. automation-sdet-agent
**Role**: Test Automation Engineering
**Stack**: Playwright, Pytest, CI/CD integrations
**Use For**: Building automated UI/API test suites, CI test pipeline setup, regression automation
**NOT For**: Manual/exploratory testing, application code

**Timing**: Engage when test cases are stable and repeatable (typically after manual validation confirms they're worth automating).

---

## Assignment Decision Matrix

| Need | Assign |
|---|---|
| Coordinate multiple agents / project management | hiring-manager-orchestrator |
| Requirements are unclear | business-analyst-agent |
| UI only | frontend-engineer |
| Backend/API/DB only | backend-engineer |
| Full feature (UI + API + DB) | full-stack-engineer |
| Infrastructure/deployment/CI | devops-cloud-engineer |
| LLM, chatbot, RAG, prompt engineering | genai-llm-engineer |
| Data pipeline, ML model | data-ml-engineer |
| Manual testing / QA sign-off | manual-functional-sdet |
| Automated test suite / CI tests | automation-sdet-agent |

## Delivery Lifecycle

```
/start-project → BA readiness → /design-doc (M/L/XL + ambiguous items) → /sprint-plan → dev → QA → release
```

Quality gates in sequence:
1. **BA gate** (`/sprint-plan` Step 3) — blocks `needs-BA` items; BA must set `BA Status = ready` first
2. **Ambiguity gate** (`/sprint-plan` Step 5) — any item with Ambiguity Medium/High requires `/design-doc`, even Small items
3. **Design approval gate** (`/design-doc` Section 9) — human non-author approval required; `/task-start` blocks until `design-doc: done`
4. **DoR gate** (`/task-start` Step 2) — measurable criteria, BA Status, ambiguity resolved
5. **Code review gate** (`/code-review`) — before READY FOR QA
6. **QA gate** (`manual-functional-sdet`) — before DONE
7. **Release gate** (`/release-checklist`) — reads SPRINT.md + TASK.md + BUG-N.md

## Parallel vs Sequential Patterns

**Always Sequential**:
- BA (requirements + SPRINT.md updates) → design docs → `/sprint-plan` → Dev → Manual QA → Release
- API design (backend) → Frontend integration

**Can Be Parallel**:
- Backend API design + DevOps environment setup
- Data pipeline work + Frontend dashboard work (if API contract is agreed)
- Manual QA on feature A + Dev working on feature B
