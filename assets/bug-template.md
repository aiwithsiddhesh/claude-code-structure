# BUG-{N}: {short descriptive title}

## Identity

| Field | Value |
|---|---|
| **ID** | BUG-{N} |
| **Status** | OPEN |
| **Severity** | {Critical / High / Medium / Low} |
| **Type** | {Functional / Security / Performance / UI / Data} |
| **Project** | {project} |
| **Opened** | {today} |
| **Assigned Agent** | {dev agent} |
| **Sprint** | Sprint {N} |
| **Discovered By** | {developer self-found / QA / stakeholder} |
| **Frequency** | {Always / Intermittent / Once} |

---

## Reproduction Steps

[Exact steps to reproduce. Must be complete enough that another agent can follow them without ambiguity.]

1. {Step 1}
2. {Step 2}
3. {Step 3}

**Environment**: {environment where bug was observed}

---

## Expected Behaviour

[What should happen when following the reproduction steps above.]

---

## Actual Behaviour

[What actually happens. Include error messages, incorrect output, or evidence.]

---

## Root Cause Hypothesis

[Initial hypothesis about what is causing the bug. Write "Unknown — requires investigation" if not yet known.]

---

## Acceptance Criteria for Fix

[Each criterion must be measurable and independently verifiable. These replace sprint task criteria — this is the dev agent's definition of done.]

- [ ] {Criterion 1 — e.g., "Login succeeds with valid credentials on all supported browsers"}
- [ ] {Criterion 2 — e.g., "Original reproduction steps no longer reproduce the bug"}
- [ ] {Criterion 3 — e.g., "No regression in adjacent features [list them]"}

---

## Fix

[Filled in by the dev agent after the fix is complete. Leave blank until then.]

**Files changed**:
| File | Change Description |
|---|---|
| {path} | {what was changed and why} |

**Fix description**: [What was changed and why the bug occurred.]

---

## Verification

[Filled in by manual-functional-sdet running /bug-verify. Leave blank until then.]

**Status**: [VERIFIED / REJECTED]
**Agent**: manual-functional-sdet
**Date verified**: {date}
**Result**: [VERIFIED / REJECTED]
**Notes**: [what was tested]

**Acceptance criteria results**:
- [ ] {Criterion 1} — [Pass / Fail]
- [ ] {Criterion 2} — [Pass / Fail]

**Regression check**: [areas checked and result]

---

## Regression Test

[Filled in by automation-sdet-agent after verification. Leave blank until then.]

**Status**: [Ready for automation / In progress / Done]
**Agent**: automation-sdet-agent
**Test file**: {path to test file}
**Test name**: {test name}
**Naming convention**: `bug-{N}-{short-description}.test.{ext}`
**CI status**: [passing / failing / not yet run]
