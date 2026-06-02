# BUG-{N} — {short descriptive title}

**Status**: OPEN
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

## Verification

**Status**: Pending fix completion
**Agent**: manual-functional-sdet
**Skill**: `/bug-verify {project} BUG-{N}` when fix is READY FOR VERIFICATION
**Date verified**: 
**Result**: VERIFIED / REJECTED
**Notes**: 

---

## Regression Test

**Status**: Pending verification
**Agent**: automation-sdet-agent
**Triggered by**: BUG-{N}.md status moving to VERIFIED
**Test file**: 
**Test name**: 
**CI status**: 
