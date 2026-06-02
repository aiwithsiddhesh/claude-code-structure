---
name: design-doc
description: Generate a technical design document for a named feature or system. Use before development begins on any non-trivial feature to force explicit thinking about the approach and surface risks early. Reads feature name from $ARGUMENTS and any existing requirements context. Produces a design document covering problem statement, scope, solution architecture, API contract, DB changes, alternatives, risks, and testing plan.
disable-model-invocation: true
argument-hint: "[feature name]"
---

# Technical Design Document: $ARGUMENTS

Write a complete technical design document before development begins. Force explicit thinking about the approach, surface risks early, and reach team alignment.

---

## Step 1 — Gather Context

Before filling the template, read:
- The feature requirements and acceptance criteria from SPRINT.md (if a sprint is active)
- Any existing related code, APIs, or DB schemas this feature will touch
- Any constraints from `project-intake.json` that apply to this feature

---

## Step 2 — Fill the Design Document

Use the template at `assets/design-doc-template.md`. Fill in all `[placeholder]` values:
- `[Feature Name]` — from $ARGUMENTS
- `[today]` — current date
- `[agent/person]` — your agent name
- All stakeholders, scope, solution, API contract, DB changes, risks, and testing plan — from context gathered in Step 1

Do not leave any section empty. If a section is not applicable (e.g., no DB changes), write "Not applicable — [reason]".

---

## Step 3 — Output and Confirm

Output the completed design document.

Then confirm:
```
✅ Design document generated for {feature name}.
Review open questions (Section 5) and resolve with owners before development starts.
Next: /handoff business-analyst-agent to [lead engineer] once the design is approved, or assign directly to the development agent.
```
