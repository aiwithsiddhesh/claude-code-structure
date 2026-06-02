# Agent Coordination Rules

## Assignment Decision Matrix

| Need | Agent |
|---|---|
| Project coordination, blockers, release | hiring-manager-orchestrator |
| Requirements, user stories, acceptance criteria | business-analyst-agent |
| UI, React, frontend, forms, accessibility | frontend-engineer |
| APIs, databases, auth, backend logic | backend-engineer |
| Cross-stack feature (UI + API + DB) | full-stack-engineer |
| Infra, CI/CD, Docker, deployments, environments | devops-cloud-engineer |
| LLM, RAG, chatbot, prompts, embeddings | genai-llm-engineer |
| Data pipelines, ML models, predictions | data-ml-engineer |
| Manual testing, bug verification, regression | manual-functional-sdet |
| Automated tests, CI test suites | automation-sdet-agent |

## Self-Limitation Rules

- **hiring-manager-orchestrator** does not code, test, or do deep technical work. It coordinates.
- **genai-llm-engineer** and **data-ml-engineer** are only assigned when the project genuinely involves AI/ML. Do not default to them for general engineering.
- **full-stack-engineer** is used when a single feature spans both frontend and backend; do not assign both frontend-engineer AND backend-engineer for the same feature unless the work can be cleanly parallelized.

## Parallel vs Sequential Work

Use **parallel assignment** when:
- Tasks have no interdependencies (e.g., backend API design + infrastructure setup)
- Research can happen concurrently (e.g., requirements + tech spike)

Use **sequential assignment** when:
- Output of one agent is input to another (e.g., BA writes requirements → backend designs API)
- Tests must be written after implementation (QA after dev, not before)

## QA Involvement Timing

- **manual-functional-sdet** — Engaged after dev completes and `/code-review` APPROVED is recorded in TASK.md. Validates feature against acceptance criteria.
- **automation-sdet-agent** — Self-triggered from `manual-functional-sdet` output. Does NOT wait for orchestrator assignment. Picks up test cases marked `AUTOMATION READY` in manual QA notes.

**A test case is AUTOMATION READY when ALL of the following are true:**
1. The feature has passed manual QA at least once (TASK.md status is `DONE` or bug shows `VERIFIED`)
2. The test case has clear, documented reproduction steps with no ambiguity
3. The test case passed on two consecutive manual runs without changes to steps
4. The acceptance criteria have not changed in the current sprint (no open change requests affecting this feature)
5. The feature is not scheduled for significant rework in the next sprint

**Automation is NOT appropriate for:**
- Features with task status `IN PROGRESS`, `BLOCKED`, or `REWORK`
- Test cases that depend on exploratory steps or human judgment
- One-off validation tests that will not recur
- Features with open change requests that could alter acceptance criteria

Never release without QA sign-off from manual-functional-sdet.
