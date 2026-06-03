# BUG-{N} — {short descriptive title}

**Status**: OPEN
<!-- Valid status values: OPEN | IN FIX | READY FOR VERIFICATION | REJECTED | VERIFIED | CLOSED
     State machine: OPEN → IN FIX → READY FOR VERIFICATION → VERIFIED → CLOSED
                                                            ↓           ↑
                                                         REJECTED ──────┘
     REJECTED means fix failed verification; dev resumes with /task-resume or /task-start, then sets IN FIX when actively working again.
     A bug is not fixed until CLOSED. VERIFIED means fix works but regression test not yet in CI. -->
**Severity**: {Critical / High / Medium / Low}
**Type**: {Functional / Security / Performance / UI / Data}
**Project**: {project}
**Sprint at discovery**: Sprint {N}
**Found by**: {agent or stakeholder}
**Assigned to**: {dev agent}
**Date**: {today}

---

## Reproduction Steps

1. {exact step}
2. {exact step}
3. ...

**Frequency**: Always / Intermittent / Once
**Environment**: {browser, OS, env — fill in if known}

## Expected Behaviour
{what should happen}

## Actual Behaviour
{what actually happens}

## Root Cause Hypothesis
{initial guess — assigned agent will confirm or revise}

---

## Acceptance Criteria for Fix

- [ ] {specific condition that confirms the bug is gone}
- [ ] {specific condition for adjacent behaviour that must not regress}
- [ ] Regression test added by automation-sdet-agent

---

## Fix

**Status**: Not started
**Agent**: {assigned dev agent}
**Start**: Use `/task-start {project} BUG-{N}` — treat this as a task, acceptance criteria above are your DoD
**Files changed**: (filled in when fix is complete)
**Fix description**: (filled in when fix is complete)

---

## Verification History

Record every verification attempt here. Never overwrite — append.

### Attempt 1

**Date**: (filled in by manual-functional-sdet)
**Agent**: manual-functional-sdet
**Result**: VERIFIED / REJECTED
**Reason** (if REJECTED): {exactly what still fails — which acceptance criterion, what evidence}
**Evidence**: {specific output, error message, or observation}
**Affected criteria**: {which criteria failed}
**Resubmission instructions**: {what the dev agent must fix before resubmitting}

---

## Regression Test

**Status**: Pending verification
**Agent**: automation-sdet-agent
**Triggered by**: BUG-{N}.md status moving to VERIFIED
**Test file**: 
**Test name**: 
**CI status**: 
