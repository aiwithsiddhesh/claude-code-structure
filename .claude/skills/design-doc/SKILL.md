---
description: Generate a technical design document for a feature or system. Use before development begins on any non-trivial feature to align on approach before writing code.
disable-model-invocation: true
argument-hint: "[feature name]"
---

# Technical Design Document: $ARGUMENTS

Generate a complete technical design document. This document should be written BEFORE development begins, to force explicit thinking about the approach, surface risks early, and get team alignment.

---

## Document Template

```markdown
# Technical Design: [Feature Name]

**Author**: [agent/person]
**Date**: [today]
**Status**: Draft | In Review | Approved
**Stakeholders**: [list agents/people who should review]

---

## 1. Problem Statement

**What problem are we solving?**
[1–3 sentences. Be specific. Include who is affected and what happens today without this feature.]

**Why now?**
[Business or technical driver. What changes if we don't build this?]

**Success criteria:**
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

---

## 2. Scope

**In scope**:
- [What this design covers]

**Out of scope** (explicitly):
- [What this design does NOT cover — important to prevent scope creep]

**Assumptions**:
- [What must be true for this design to work]

---

## 3. Proposed Solution

**Overview** (2–4 sentences summarizing the approach):

**Architecture / Data Flow**:
[Diagram or step-by-step description of how data moves through the system]

```
[User] → [Frontend] → [API /endpoint] → [Service] → [Database]
                                      ↓
                               [External service]
```

**Key Components**:

| Component | Responsibility | New or Existing |
|---|---|---|
| [component] | [what it does] | New / Existing |

**API Contract** (if applicable):
```
POST /api/v1/[resource]
Request: { ... }
Response: { data: { ... } }
```

**Database Changes** (if applicable):
- New table: `[table_name]` with columns: [list]
- New column: `[table].[column]` type [type], nullable/not null
- New index: `[description]`
- Migration reversible: Yes / No (explain if No)

---

## 4. Alternatives Considered

For each alternative you ruled out, explain why:

| Alternative | Why Rejected |
|---|---|
| [approach] | [reason] |
| [approach] | [reason] |

---

## 5. Risks & Mitigations

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| [risk] | H/M/L | H/M/L | [how to mitigate] |

**Open questions** (must be resolved before development starts):
- [ ] [Question] — Owner: [person]
- [ ] [Question] — Owner: [person]

---

## 6. Implementation Plan

**Agent assignments**:
| Agent | Tasks |
|---|---|
| [agent] | [tasks] |

**Sequence**:
1. [Step 1]
2. [Step 2]
3. ...

**Estimated complexity**: Small (< 1 day) / Medium (1–3 days) / Large (3–7 days) / XL (> 1 week)

---

## 7. Testing Plan

| Test Type | Coverage |
|---|---|
| Unit tests | [what will be unit tested] |
| Integration tests | [what will be integration tested] |
| Manual QA | [key scenarios for manual-functional-sdet] |
| Performance | [if applicable] |

---

## 8. Rollout Plan

- **Feature flag**: Yes / No
- **Rollback plan**: [how to roll back if something goes wrong]
- **Monitoring**: [what metrics/alerts to watch after deployment]

---

## Review Checklist

Before approving this design, reviewers should confirm:
- [ ] Problem statement is clear and correct
- [ ] Scope boundaries are explicit
- [ ] API contract is complete
- [ ] Database changes are migration-safe
- [ ] Security implications addressed (auth, input validation, PII)
- [ ] All open questions have owners
- [ ] Test plan covers acceptance criteria
```
