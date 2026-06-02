---
name: code-review
description: Run a structured code review on a completed sprint task before it moves to QA. Covers correctness, security, code quality, test coverage, and documentation. Must be run by every coding agent before marking a task complete. Output is APPROVED or CHANGES REQUESTED — result must be recorded in TASK.md Completion entry.
disable-model-invocation: true
argument-hint: "[project-name] [task-id]"
---

# Code Review — $ARGUMENTS

## Read Task State

```
!`cat output/$(echo $ARGUMENTS | cut -d' ' -f1)/.tasks/$(echo $ARGUMENTS | cut -d' ' -f2).md 2>/dev/null || echo "ERROR: TASK.md not found. Run /task-start {project} {task-id} first."`
```

---

You are the agent who wrote this code, or a same-domain peer reviewer. Work through every section. Do not skip any checklist item.

---

## Step 1 — Validate Preconditions

- TASK.md must exist. If not → `❌ No TASK.md found. Run /task-start {project} {task-id} first.`
- All steps in TASK.md must be checked off. If any are unchecked → `❌ Task {task-id} has unchecked steps. Complete all steps before running code review.`
- Confirm reviewer identity:
  - **Self-review**: you are the agent who wrote the code. Valid for solo projects.
  - **Peer review**: you are a different agent in the same domain (e.g., a second `frontend-engineer` reviewing `frontend-engineer` code). Valid for team projects.
  - `hiring-manager-orchestrator` may not be a code reviewer under any circumstances.

---

## Step 2 — Run Automated Checks

Confirm that all automated checks have been run and pass. Record the result for each:

| Check | TypeScript/JavaScript | Python | Status |
|---|---|---|---|
| Linter | `ESLint` — zero errors | `ruff` or `flake8` — zero errors | ✅ / ❌ |
| Type checker | `tsc --noEmit` — zero errors | `mypy` — zero errors | ✅ / ❌ |
| Security scan | `npm audit` — no Critical/High CVEs | `pip-audit` — no Critical/High CVEs | ✅ / ❌ |

If any check fails → **do not continue**. Fix the issue and re-run the check before proceeding with the review.

---

## Step 3 — Correctness Review

For each acceptance criterion in TASK.md, verify the code satisfies it:

```
Criterion 1: {text}
  → Status: ✅ Satisfied / ❌ Not satisfied
  → Evidence: {file path, function name, or test that proves it}
  → Notes: {anything the reviewer noticed}

Criterion 2: {text}
  → Status: ✅ Satisfied / ❌ Not satisfied
  → Evidence: {file path, function name, or test that proves it}
  → Notes: {anything the reviewer noticed}
```

If any criterion is not satisfied → this is a blocking finding.

---

## Step 4 — Security Review

Run through the checklist from `.claude/rules/security.md`. For each item, confirm it applies to this task and whether it passes:

- [ ] No secrets, API keys, or credentials in the diff
- [ ] All user inputs validated server-side (not just client-side)
- [ ] No SQL string concatenation — parameterized queries only
- [ ] Authentication present on all protected endpoints
- [ ] Passwords and sensitive data properly hashed (never stored plaintext)
- [ ] Error responses do not expose internal details (stack traces, file paths, IDs)
- [ ] CORS configured — no wildcard `*` in non-local environments
- [ ] File paths from user input are sanitized (no `../` traversal)
- [ ] New dependencies reviewed — no known Critical/High CVEs

Mark each item `✅ Pass`, `❌ Fail`, or `N/A — not applicable to this task`. Any `❌` is a blocking finding.

---

## Step 5 — Code Quality Review

Check the code against `coding-standards.md` requirements:

- [ ] Naming conventions correct (camelCase, PascalCase, snake_case as appropriate for stack)
- [ ] No function longer than 40 lines
- [ ] No file longer than 300 lines
- [ ] No magic numbers — constants extracted with descriptive names
- [ ] No commented-out code committed
- [ ] One responsibility per function — no "and" needed to describe what it does
- [ ] Error messages are actionable (not "Invalid input" — must be specific)
- [ ] Imports ordered correctly (Node built-ins → External → Internal → Relative)
- [ ] `async/await` used over raw Promises (TypeScript/JavaScript)
- [ ] Type annotations on all function signatures (Python)

Mark each item `✅ Pass`, `❌ Fail`, or `N/A`. Any `❌` is a finding — classify as blocking or advisory.

---

## Step 6 — Test Coverage Review

- [ ] Unit tests written for all business logic functions
- [ ] Edge cases covered: empty inputs, boundary values, invalid types
- [ ] Error paths tested (what happens when the API returns 500, DB is down, input is malformed)
- [ ] No tests that rely on implementation details — tests verify behavior, not internals
- [ ] Tests can run in isolation (no shared state between tests)
- [ ] Test names describe what they verify: `it('returns 404 when user does not exist')` not `it('test user endpoint')`

Mark each item `✅ Pass`, `❌ Fail`, or `N/A`. Any `❌` is a finding.

---

## Step 7 — Documentation Review

- [ ] All exported functions/classes have JSDoc (TypeScript/JavaScript) or docstrings (Python)
- [ ] Non-obvious logic has a comment explaining WHY (not what — the code already shows what)
- [ ] Any `TODO:` comments include a ticket reference: `// TODO(#142): description`
- [ ] API endpoints are documented (request/response schema, status codes, auth requirements)
- [ ] README or inline docs updated if public interface changed

Mark each item `✅ Pass`, `❌ Fail`, or `N/A`.

---

## Step 8 — Produce Review Output

Collect all findings from Steps 3–7 and produce the review result:

```markdown
## Code Review — {task-id} — {today}

**Reviewer**: {agent name} ({self-review / peer review})
**Task**: {task description}

### Automated Checks
| Check | Result |
|---|---|
| Linter | ✅ Pass / ❌ {error count} errors |
| Type checker | ✅ Pass / ❌ {error count} errors |
| Security scan | ✅ Pass / ❌ {CVE count} Critical/High CVEs |

### Findings

**Blocking findings** (must be fixed before APPROVED):
1. {finding}: {file path, line or function, exact description of the problem}
2. {finding}: ...

**Advisory findings** (should fix but does not block approval):
1. {finding}: {description}

### Result: APPROVED / CHANGES REQUESTED

{If APPROVED}: All acceptance criteria satisfied, no blocking findings. Task may proceed to READY FOR QA.

{If CHANGES REQUESTED}: {N} blocking finding(s) must be addressed. Re-run /code-review after fixing.
```

---

## Step 9 — Record Result in TASK.md

If result is **APPROVED**:

Proceed to run `/task-checkpoint {project} {task-id}` completion flow. The Completion entry must include:
```
**Code review**: APPROVED
**Code review notes**: {brief summary — reviewer, what was checked, any advisory findings noted}
```

If result is **CHANGES REQUESTED**:

Do NOT write a Completion entry. Fix every blocking finding, then re-run `/code-review {project} {task-id}` from Step 2.

Confirm:
```
{✅ APPROVED — {task-id} ready for QA. Run /task-checkpoint to write Completion entry.}
{❌ CHANGES REQUESTED — {N} blocking finding(s). Fix and re-run /code-review.}
```
