# Design Doc: [Feature Name]

**Author**: [agent/person]
**Date**: [today]
**Status**: DRAFT | REVIEW | APPROVED
**Feature**: [Feature Name]
**Stakeholders**: [list stakeholders, e.g. hiring-manager-orchestrator, product owner]

---

## Section 1 — Problem Statement

[Describe the problem this feature solves. Be specific about who is affected and what the impact is.]

---

## Section 2 — Scope

### In Scope
- [Explicit list of what this design covers]

### Out of Scope
- [Explicit list of what this design does NOT cover — reference project-intake out_of_scope if applicable]

---

## Section 3 — Proposed Solution

### Architecture Overview

```
[ASCII diagram or description of component relationships]
```

### Key Design Decisions
| Decision | Choice | Reason |
|---|---|---|
| [e.g., Storage layer] | [e.g., PostgreSQL] | [reason] |

### Component Breakdown
[Describe each major component or module. One paragraph per component.]

---

## Section 4 — API Contract

| Method | Path / Operation | Request | Response | Auth |
|---|---|---|---|---|
| [GET] | [/api/v1/example] | [params] | [shape] | [required/none] |

[If no API changes: "Not applicable — this feature does not add or change API endpoints."]

---

## Section 4b — Database Changes

| Change | Table | Description |
|---|---|---|
| [ADD COLUMN] | [table_name] | [column_name type — reason] |

[If no DB changes: "Not applicable — this feature does not require schema changes."]

---

## Section 5 — Risks & Mitigations

### Risk Table
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| [e.g., Third-party API rate limiting] | Medium | High | [Cache responses, add retry with backoff] |

### Alternatives Considered
| Alternative | Why Not Chosen |
|---|---|
| [Option A] | [Reason rejected] |

### Open Questions
[Each unresolved question must be a checkbox. All must be checked before development starts.]

- [ ] [Question 1 — assign an owner: @agent-name]
- [ ] [Question 2 — assign an owner: @agent-name]

[If no open questions at time of writing: remove the checkbox items and write "No open questions — design is ready for development."]

---

## Section 6 — Testing Plan

| Test Type | What is Tested | Owner |
|---|---|---|
| Unit | [specific function or class] | [dev agent] |
| Integration | [specific interaction, e.g., API → DB] | [dev agent] |
| Manual QA | [acceptance criteria from SPRINT.md] | manual-functional-sdet |
| Automation | [which manual tests become regression suite] | automation-sdet-agent |

---

## Section 7 — Review Checklist

- [ ] Problem statement is clear and agreed by stakeholders
- [ ] Scope boundaries are explicit
- [ ] Architecture diagram reviewed
- [ ] API contract agreed by all consumers
- [ ] DB migration is backward-compatible or migration plan documented
- [ ] All Section 5 open questions resolved (no unchecked `- [ ]` items)
- [ ] Testing plan covers all acceptance criteria
- [ ] Security implications reviewed against `.claude/rules/security.md`
