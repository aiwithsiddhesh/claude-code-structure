---
name: bug-verify
description: Verify a bug fix. Run by manual-functional-sdet after a dev agent marks a bug READY FOR VERIFICATION. Re-tests reproduction steps, validates acceptance criteria, and updates BUG-{N}.md to VERIFIED or REJECTED.
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

Update the **Verification** section of BUG-{N}.md:
```markdown
## Verification

**Status**: VERIFIED
**Agent**: manual-functional-sdet
**Date verified**: {today}
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
**Action required**: Write regression test based on reproduction steps above. Mark CLOSED when CI passes.
```

Update SPRINT.md Bug Log — change status from `READY FOR VERIFICATION` to `VERIFIED`.

Output:
```
✅ BUG-{N} VERIFIED.
Assigned to automation-sdet-agent for regression test.
automation-sdet-agent: read output/{project}/.bugs/BUG-{N}.md
and write a regression test for the reproduction steps. 
Mark CLOSED when the test passes in CI.
```

---

**If REJECTED:**

Update the **Verification** section of BUG-{N}.md:
```markdown
## Verification

**Status**: REJECTED — returned to {dev agent}
**Agent**: manual-functional-sdet
**Date**: {today}
**Result**: REJECTED
**Reason**: {exactly what still fails and which acceptance criterion was not met}
**Evidence**: {specific output, error message, or screenshot description}

**To resubmit**: Fix the issue described above, update the Fix section,
set status back to READY FOR VERIFICATION, then /bug-verify {project} BUG-{N} again.
```

Update the **Status** field at the top: `IN FIX`
Update SPRINT.md Bug Log — change status back to `IN FIX`.

Output:
```
❌ BUG-{N} REJECTED — returned to {dev agent}.
Reason: {reason}
Dev agent must address the rejection reason and resubmit for verification.
```
