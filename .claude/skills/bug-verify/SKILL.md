---
name: bug-verify
description: Verify a bug fix. Run by manual-functional-sdet after a dev agent marks a bug READY FOR VERIFICATION. Re-tests reproduction steps, validates acceptance criteria, and updates BUG-{N}.md to VERIFIED or REJECTED. On REJECTED, sets status to REJECTED (not IN FIX) — dev agent must resume work to move it back to IN FIX.
disable-model-invocation: true
argument-hint: "[project-name] [bug-id]"
---

# Bug Verify — $ARGUMENTS

## Read Bug State

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/.bugs/$(echo $ARGUMENTS | cut -d' ' -f2).md 2>/dev/null || echo "ERROR: $(echo $ARGUMENTS | cut -d' ' -f2).md not found in output/$(echo $ARGUMENTS | cut -d' ' -f1)/.bugs/"`
```

---

You are **manual-functional-sdet**. Verify this bug fix before it can be marked closed.

---

## Step 1 — Validate Preconditions

- BUG-{N}.md must exist. If not → `❌ Bug file not found. Was /bug-triage run?`
- Bug status must be `READY FOR VERIFICATION`. If status is:
  - `OPEN` or `IN FIX` → `❌ Fix not complete. Dev agent must finish the fix and update BUG-{N}.md status to READY FOR VERIFICATION first.`
  - `REJECTED` → `❌ This bug was previously rejected. Dev agent must resume work (/task-resume or /task-start), fix the issues described in the Verification History, set status to IN FIX when actively working, then READY FOR VERIFICATION when done.`
  - `VERIFIED` → `✅ Already verified. automation-sdet-agent should write regression test if not done.`
  - `CLOSED` → `✅ Already closed. No action needed.`

Check that the **Fix** section of BUG-{N}.md is filled in:
- Files changed must be listed
- Fix description must be written
If either is empty → return to dev agent: `❌ Fix section incomplete. {agent} must document what was changed before verification.`

---

## Step 2 — Re-run Reproduction Steps

Execute the exact reproduction steps from BUG-{N}.md in order.

For each step, record:
- Did it execute as described?
- What was the result?

**If the original bug still occurs:**
→ Verification result is REJECTED. Go to Step 4 (rejection flow).

**If the original bug no longer occurs:**
→ Continue to Step 3.

---

## Step 3 — Validate Acceptance Criteria

Check each acceptance criterion from BUG-{N}.md:

| Criterion | Test | Result |
|---|---|---|
| {criterion 1} | {how tested} | Pass / Fail |
| {criterion 2} | {how tested} | Pass / Fail |

**Regression check** — test adjacent behaviour that could have been broken by the fix:
- What other features use the same code path that was changed?
- Run a quick sanity check on those areas.
- If any regression found → treat as new bug, run `/bug-triage {project}` for it.

If all criteria pass and no regressions found → proceed to VERIFIED flow.
If any criterion fails → proceed to REJECTED flow.

---

## Step 4 — Update BUG-{N}.md

**If VERIFIED:**

Append to the **Verification History** section of BUG-{N}.md (do not overwrite existing attempts):
```markdown
### Attempt {N} — VERIFIED

**Date**: {today}
**Agent**: manual-functional-sdet
**Result**: VERIFIED
**Notes**: {brief notes on what was tested}

**Acceptance criteria results**:
- [x] {criterion 1} — Pass
- [x] {criterion 2} — Pass

**Regression check**: {areas checked, no regressions found}
```

Update the **Status** field at the top: `VERIFIED`

Update the **Regression Test** section:
```markdown
## Regression Test

**Status**: Ready for automation
**Agent**: automation-sdet-agent
**Action required**: Write regression test based on reproduction steps above. When the test passes in CI, set BUG-{N}.md Status to CLOSED and update SPRINT.md Bug Log from VERIFIED to CLOSED.
```

Update SPRINT.md Bug Log — change status from `READY FOR VERIFICATION` to `VERIFIED`.

Output:
```
✅ BUG-{N} VERIFIED.
Assigned to automation-sdet-agent for regression test.
automation-sdet-agent: read output/{project}/.bugs/BUG-{N}.md and write a regression test for the reproduction steps.
When the test passes in CI:
1. Set BUG-{N}.md Status to `CLOSED`
2. **Update SPRINT.md Bug Log — change status from `VERIFIED` to `CLOSED`**
3. Confirm: `✅ BUG-{N} CLOSED. Regression test passing in CI at {test file path}.`
```

---

**If REJECTED:**

Append to the **Verification History** section of BUG-{N}.md (do not overwrite — each attempt is a permanent record):
```markdown
### Attempt {N} — REJECTED

**Date**: {today}
**Agent**: manual-functional-sdet
**Result**: REJECTED
**Reason**: {exactly what still fails — be specific, not generic}
**Evidence**: {specific output, error message, or observation that proves the failure}
**Affected criteria**: {which acceptance criteria failed}
**Owner**: {dev agent assigned to this bug}

**Resubmission instructions**:
1. {Specific fix required — what exactly must change}
2. {Any other steps needed}
3. Set Fix section to updated description of what was changed
4. Set Status to IN FIX when actively working
5. Set Status to READY FOR VERIFICATION when fix is complete
6. Re-run /bug-verify {project} {bug-id}
```

Update the **Status** field at the top: `REJECTED`

Update SPRINT.md Bug Log — change status to `REJECTED`.

Output:
```
❌ BUG-{N} REJECTED — returned to {dev agent}.

Reason: {specific reason}
Affected criteria: {which criteria failed}
Evidence: {specific observation}

Dev agent next steps:
1. If a TASK.md already exists for BUG-{N}: manually set BUG-{N}.md Status to `IN FIX`, then run /task-resume {project} BUG-{N} to resume work
2. If no TASK.md exists: run /task-start {project} BUG-{N} to begin the fix
3. Fix the issues documented in Verification History Attempt {N}
4. Set BUG-{N}.md Status to IN FIX when actively working
5. Set Status to READY FOR VERIFICATION when complete
6. Run /bug-verify {project} BUG-{N} again

Status is now REJECTED. Dev agent must explicitly set to IN FIX when work resumes.
```
