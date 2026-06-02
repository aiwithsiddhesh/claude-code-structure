---
name: release-checklist
description: Run the pre-release quality gate checklist before any production deployment. Use after all sprint features are complete and QA has signed off, as the final gate before going to production. Reads [feature or release name] from $ARGUMENTS. Produces a GO / NO-GO / CONDITIONAL GO decision with sign-off covering development completeness, testing, documentation, infrastructure, security, and release readiness.
disable-model-invocation: true
argument-hint: "[project-name]"
---

# Release Checklist: $ARGUMENTS

## Read Current State

```
!`cat output/$ARGUMENTS/SPRINT.md 2>/dev/null || echo "ERROR: output/$ARGUMENTS/SPRINT.md not found. Verify the project name."`
```

---

## Step 1 — Validate Preconditions

Before running the checklist, confirm:
- SPRINT.md must exist for this project. If not found → `❌ Cannot run release checklist — no project state found. Check project name.`
- All committed sprint items should be DONE or explicitly deferred. If any item is IN PROGRESS, BLOCKED, or REWORK → flag it: `⚠️ Sprint item {ID} is {status}. Confirm it is intentionally excluded from this release before proceeding.`
- QA sign-off from `manual-functional-sdet` must exist for all items in scope. If not confirmed → `❌ Cannot release — manual-functional-sdet sign-off not found for {item}.`

If preconditions are not met, stop and report what must be resolved before re-running.

---

Work through each category below. Mark ✅ Pass, ❌ Fail, or ⚠️ Needs Review for each item.

## Development Completeness
- [ ] All acceptance criteria implemented
- [ ] Code reviewed (peer review or self-review documented)
- [ ] No known critical or high-severity bugs open
- [ ] All TODO/FIXME comments addressed or tracked
- [ ] Feature flags set correctly for release

## Testing
- [ ] Manual QA completed by manual-functional-sdet
- [ ] QA sign-off received (not self-certified by dev)
- [ ] Regression tests pass
- [ ] Automated tests pass in CI pipeline
- [ ] Edge cases and error states tested

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

**Sign-off**: [Agent/person completing this checklist]
**Date**: [today]
