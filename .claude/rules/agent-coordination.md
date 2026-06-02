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

- **manual-functional-sdet** — Engaged after dev completes; validates feature against acceptance criteria
- **automation-sdet-agent** — Engaged when test cases are stable and repeatable; builds regression suite

Never release without QA sign-off from manual-functional-sdet.
