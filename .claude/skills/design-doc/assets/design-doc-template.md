# Technical Design: [Feature Name]

**Author**: [agent/person]
**Date**: [today]
**Status**: Draft | In Review | Approved | BLOCKED
**Stakeholders**: [list agents/people who should review]
**SPRINT.md sync**: Not yet synced | Synced — design-doc: done | Synced — design-doc: blocked

---

## 0. Ambiguity Assessment

**Complete this section before any other section.** Identify every unknown, gap, and decision that must be resolved before development can start. Be specific — do not template-fill your way through this.

### Missing information (must be answered before design is complete)

- [ ] [Question] — Owner: [person/agent] — Required decision: [what must be decided]
- [ ] [Question] — Owner: [person/agent] — Required decision: [what must be decided]

### Missing API decisions

- [ ] [Which endpoints? What request/response shape? Who owns the contract?]

### Missing data/schema decisions

- [ ] [Which tables? What columns? Migration strategy? Rollback?]

### Missing security decisions

- [ ] [Auth model? PII handling? Input validation scope?]

### Unresolved tradeoffs

- [ ] [Option A vs Option B — who decides and by when?]

### Stakeholder/product questions

- [ ] [Questions that require product owner or stakeholder input before dev starts]

**Ambiguity level**: Low (0 open items) | Medium (1–3 open items) | High (4+ open items)

If Ambiguity is Medium or High, this design MUST be resolved before `/task-start` may proceed, regardless of item Complexity.

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

## 9. Approval Record

This section must be filled in by a human, product owner, or lead engineer before this design is marked Approved. The author may not self-approve.

**Decision required**: Should this design proceed to development as written?

**Options considered**:
1. [Option A — e.g., proceed as designed]
2. [Option B — e.g., alternative approach X]
3. [Option C — e.g., defer / descope]

**Chosen option**: [which option was selected]

**Rationale**: [why this option over the alternatives — must be specific, not "seemed best"]

**Approver**: [name/role — must be a human, product owner, or lead engineer; not the author]

**Date approved**: [date]

**Conditions / caveats**: [any conditions attached to approval, or "None"]

> If this section is blank or filled by the author themselves, the design is NOT approved.
> Run `/design-doc [feature-name]` to re-evaluate after getting approval.

---

## 10. Review Checklist

Before marking this design Approved, the approver should confirm:
- [ ] Section 0 Ambiguity Assessment is complete — all items resolved or explicitly accepted
- [ ] Problem statement is clear and correct
- [ ] Scope boundaries are explicit
- [ ] API contract is complete (or marked N/A with reason)
- [ ] Database changes are migration-safe (or marked N/A with reason)
- [ ] Security implications addressed (auth, input validation, PII)
- [ ] All open questions in Section 0 have owners and resolution notes
- [ ] Test plan covers all acceptance criteria
- [ ] Section 9 Approval Record is filled in by a non-author approver
