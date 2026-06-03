---
name: design-doc
description: Generate a technical design document for a named feature or system. Required before development begins on any Medium/Large/XL item, and on any item with Ambiguity Medium/High regardless of size. Reads feature name from $ARGUMENTS. Produces a design document covering ambiguity assessment, problem statement, scope, solution architecture, API contract, DB changes, alternatives, risks, testing plan, and a human approval record. Updates SPRINT.md design-doc state on completion or block.
disable-model-invocation: true
argument-hint: "[feature name]"
---

# Technical Design Document: $ARGUMENTS

Write a complete technical design document before development begins. Force explicit identification of ambiguity, surface missing decisions, and reach alignment before any code is written.

---

## Step 1 — Gather Context

Before filling the template, read:
- The feature requirements and acceptance criteria from SPRINT.md (if a sprint is active)
- Any existing related code, APIs, or DB schemas this feature will touch
- Any constraints from `project-intake.json` that apply to this feature
- Any existing design doc for this feature (check `output/{project}/design-docs/` if a project directory exists)

If an existing design doc is found: update it rather than creating a new one. Apply these rules when updating:
- **Carry forward** all resolved Section 0 items (checked `[x]`) — do not uncheck them.
- **Carry forward** Section 9 (Approval Record) exactly as written — never blank, overwrite, or re-template it. If Section 9 is already filled with a named approver, that approval is preserved on re-run.
- Only re-open Section 0 items that have genuinely changed (new information contradicts a prior resolution).
- Only update sections where new context from SPRINT.md or the codebase changes the answer.

---

## Step 2 — Perform Active Ambiguity Assessment (Section 0)

**This step is mandatory and cannot be skipped or abbreviated.**

Before filling any other section, actively identify:

1. **Missing API decisions** — Are all endpoints defined? Are request/response shapes agreed? Is there an API contract between frontend and backend?
2. **Missing data/schema decisions** — Are table structures known? Is the migration strategy defined? Is rollback possible?
3. **Missing security decisions** — Is the auth model defined? Is PII handled? Is input validation scope agreed?
4. **Unresolved tradeoffs** — Are there competing approaches with no decision made? Who decides and by when?
5. **Stakeholder/product questions** — Are there requirements that depend on product owner decisions not yet made?

For each gap found, add a `- [ ]` item to Section 0 with an Owner and Required Decision.

**Do not template-fill Section 0 with empty or placeholder items.** A Section 0 that says "No missing decisions" when gaps exist is a process failure.

Determine **Ambiguity level**:
- `Low` — zero open items in Section 0
- `Medium` — 1–3 open items
- `High` — 4+ open items

---

## Step 3 — Fill the Design Document

Use the template at `assets/design-doc-template.md`. Fill in all `[placeholder]` values:
- `[Feature Name]` — from $ARGUMENTS
- `[today]` — current date
- `[agent/person]` — your agent name
- All sections — from context gathered in Steps 1–2

Do not leave any section empty. If a section is not applicable (e.g., no DB changes), write "Not applicable — [reason]".

Section 9 (Approval Record): on a **first run** (no existing doc), leave it blank — it is filled by a human, product owner, or lead engineer, not by the author. On a **re-run** (existing doc found in Step 1), carry Section 9 forward exactly as written — never blank or overwrite it.

---

## Step 4 — Evaluate and Output

After filling the document, evaluate its state:

### Check A — Ambiguity Assessment

Count all unchecked `- [ ]` items in **Section 0** (Ambiguity Assessment).

If any unchecked items exist in Section 0:

```
⚠️ DESIGN BLOCKED — {N} unresolved item(s) in Section 0 (Ambiguity Assessment).

Each item must be resolved before development starts.
Steps to unblock:
1. Assign an owner to each [ ] item in Section 0 if not already assigned.
2. Resolve each question with the owner.
3. Mark resolved items [x] with a one-line resolution note.
4. Re-run /design-doc {feature name} to re-evaluate.

Do NOT run /task-start or begin any development until all Section 0 items are resolved.
```

Go to Step 5 (SPRINT.md sync — BLOCKED state) and stop.

### Check B — Approval Record

If Section 0 is fully resolved (no unchecked items), check Section 9 (Approval Record):

If Section 9 is blank or shows no named approver:

```
⚠️ DESIGN PENDING APPROVAL — Ambiguity resolved, awaiting human approval.

Section 9 (Approval Record) must be completed by a human, product owner, or lead engineer.
The author may not self-approve.

Required fields in Section 9:
- Decision: which option was chosen
- Options Considered: at least 2 options listed
- Chosen Option: which was selected and why
- Rationale: specific reason (not "seemed best")
- Approver: named person/role (not the author)
- Date approved: date

Once Section 9 is complete, re-run /design-doc {feature name} to confirm approval and sync SPRINT.md.
```

Go to Step 5 (SPRINT.md sync — pending approval state) and stop.

### Check C — Full Approval

If Section 0 is fully resolved AND Section 9 has a named non-author approver with all required fields:

```
✅ Design document APPROVED for {feature name}.
All ambiguity resolved. Human approval recorded.
Syncing SPRINT.md...
```

Go to Step 5 (SPRINT.md sync — approved state).

---

## Step 5 — Sync SPRINT.md

**This step is mandatory every time /design-doc runs.** The design doc state must always be reflected in SPRINT.md.

First, save the design document to `output/{project}/design-docs/{feature-name}-design.md`. If the file already exists, overwrite it.

Then find the matching sprint item row in SPRINT.md and update the Design Doc column:

**If BLOCKED (unchecked Section 0 items exist):**
- Set Design Doc column to: `design-doc: blocked`
- Update the Design Decisions table with: Feature name, design doc path, Status = `blocked`, date, and a one-line blocker summary
- Update the Human Decisions table if any blocking items require product owner or stakeholder decisions

**If PENDING APPROVAL (Section 0 resolved, Section 9 blank):**
- Set Design Doc column to: `design-doc: pending-approval`
- Update the Design Decisions table with: Feature name, design doc path, Status = `pending-approval`, date

**If APPROVED (Section 0 resolved, Section 9 complete with named approver):**
- Set Design Doc column to: `design-doc: done`
- Add a link/path to the design doc file
- Update the Design Decisions table with: Feature name, design doc path, Status = `done`, Approver (from Section 9), Date (from Section 9)
- Update the Human Decisions table: record the approval decision, options considered, chosen option, rationale, decider, date — from Section 9

Confirm the sync:
```
✅ SPRINT.md updated — {feature name}: design-doc: {done | pending-approval | blocked}
Design doc saved to: output/{project}/design-docs/{feature-name}-design.md
```

If no active sprint exists (SPRINT.md has no Current Sprint), record only in the Design Decisions table and note: `Will sync to sprint item when sprint is active.`
