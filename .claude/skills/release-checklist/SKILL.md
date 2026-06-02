---
description: Run the pre-release quality gate checklist. Use before any production deployment to verify all gates are satisfied.
disable-model-invocation: true
argument-hint: "[feature or release name]"
---

# Release Checklist: $ARGUMENTS

Work through each category. Mark ✅ Pass, ❌ Fail, or ⚠️ Needs Review for each item.

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
