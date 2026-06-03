---
name: release-checklist
description: Run the pre-release quality gate checklist before any production deployment. Use after all sprint features are complete and QA has signed off, as the final gate before going to production. Reads [project-name] from $ARGUMENTS. Reads SPRINT.md, all in-scope TASK.md files, and all open BUG-N.md files to verify true completion state. Produces a GO / NO-GO / CONDITIONAL GO decision.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Release Checklist: $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found. Verify the project name."`
```

```
!`for f in output/$ARGUMENTS/.tasks/*.md; do [ -f "$f" ] && echo "=== $f ===" && cat "$f"; done 2>/dev/null || echo "NO_TASK_FILES"`
```

```
!`for f in output/$ARGUMENTS/.bugs/BUG-*.md; do [ -f "$f" ] && echo "=== $f ===" && cat "$f"; done 2>/dev/null || echo "NO_BUG_FILES"`
```

---

You are the **hiring-manager-orchestrator**. Evaluate release readiness using all three sources: SPRINT.md (sprint scope), TASK.md files (true task completion state), and BUG-N.md files (true bug status).

**Do not use SPRINT.md alone to determine release readiness.** SPRINT.md shows committed scope; TASK.md shows whether work is actually done; BUG-N.md shows whether bugs are actually closed.

---

## Step 1 — Validate Preconditions

Before running the checklist:

1. **SPRINT.md must exist.** If not found → `❌ Cannot run release checklist — no project state found. Check project name.`

2. **Read TASK.md for every in-scope sprint item.** For each item in the Current Sprint committed list:
   - Does a TASK.md exist? If not → `⚠️ No TASK.md found for {task-id}. Task may never have started or files may be missing.`
   - What is the Status field in TASK.md?
   - Is a Completion entry present (not just a checkpoint)?
   - Is code review APPROVED recorded in the Completion entry?
   - Is manual-functional-sdet sign-off recorded?
   
   If any item's TASK.md Status is `IN PROGRESS`, `BLOCKED`, `REWORK`, or `ON_HOLD` → flag it: `⚠️ Sprint item {ID} has TASK.md Status = {status}. Confirm it is intentionally excluded from this release before proceeding.`

3. **Compare SPRINT.md status vs TASK.md status for each item.** If they disagree → flag as STATUS CONFLICT and require resolution before proceeding: `❌ STATUS CONFLICT on {task-id}: SPRINT.md says {X}, TASK.md says {Y}. Resolve before running release checklist.`

4. **Read BUG-N.md for every open bug.** Check status directly from BUG-N.md, not only from the SPRINT.md Bug Log. A bug is not closed unless BUG-N.md Status = `CLOSED`. `VERIFIED` alone is not sufficient for release — regression test must be in CI (CLOSED).

5. **Check for any Critical or High bugs where BUG-N.md Status is not `CLOSED`.** If any exist → `❌ Cannot release — Critical/High bug {BUG-N} is {status} (not CLOSED). Regression test must be passing in CI.`

If preconditions are not met, stop and report what must be resolved before re-running.

---

Work through each category below. Mark ✅ Pass, ❌ Fail, or ⚠️ Needs Review for each item.

## Development Completeness
- [ ] All in-scope items have TASK.md with Status = DONE (verified from TASK.md files, not SPRINT.md alone)
- [ ] All TASK.md Completion entries present and contain code review APPROVED record
- [ ] No in-scope TASK.md shows Status IN PROGRESS, BLOCKED, REWORK, or ON_HOLD
- [ ] No known critical or high-severity bugs open (verified from BUG-N.md Status field)
- [ ] All TODO/FIXME comments addressed or tracked
- [ ] Feature flags set correctly for release

## QA Completeness
- [ ] manual-functional-sdet sign-off recorded in TASK.md Completion entry for every in-scope item
- [ ] QA sign-off is not self-certified by the dev agent
- [ ] All Critical and High bugs show BUG-N.md Status = CLOSED (regression test in CI)
- [ ] No bugs show REJECTED or READY FOR VERIFICATION status (verification incomplete)
- [ ] Edge cases and error states tested

## Automated Testing
- [ ] Regression tests pass for all VERIFIED/CLOSED bugs
- [ ] Automated test suite passes in CI pipeline
- [ ] automation-sdet-agent has closed all AUTOMATION READY test cases

## Documentation & Observability
- [ ] API documentation updated
- [ ] README or user-facing docs updated (if applicable)
- [ ] Logging and monitoring in place
- [ ] Alerts configured for new failure modes

## Infrastructure & Deployment
- [ ] Deployment pipeline tested in staging
- [ ] Environment variables and secrets configured in production
- [ ] Database migrations tested and reversible
- [ ] Rollback procedure documented and tested

## Security (if applicable)
- [ ] Input validation on all user-facing inputs
- [ ] Authentication/authorization tested
- [ ] No secrets or credentials in code
- [ ] Dependencies scanned for known vulnerabilities

## Release Readiness
- [ ] Release notes drafted
- [ ] Stakeholder communication prepared
- [ ] Deployment window confirmed
- [ ] Rollback plan defined and team is aware

---

## Checklist Result

**Release Decision**: [ ] GO  [ ] NO-GO  [ ] CONDITIONAL GO

**Conditions (if conditional)**: [list any items that must be resolved]

**Sources read**: SPRINT.md + {N} TASK.md files + {N} BUG-N.md files
**Status conflicts found**: {N} (if any, list them)

**Sign-off**: [Agent/person completing this checklist]
**Date**: [today]
