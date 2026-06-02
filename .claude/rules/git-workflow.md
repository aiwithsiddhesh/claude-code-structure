# Git Workflow

## Branch Naming

```
{type}/{short-description}
```

| Type | Use for |
|---|---|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `chore/` | Maintenance, dependencies, tooling |
| `docs/` | Documentation only |
| `refactor/` | Code changes with no behavior change |
| `test/` | Adding or fixing tests only |
| `release/` | Release preparation |

**Examples**:
- `feature/user-authentication`
- `fix/login-redirect-loop`
- `chore/upgrade-react-19`
- `docs/api-endpoint-reference`

**Rules**:
- Lowercase only. Hyphens, no underscores.
- Short and descriptive (3–5 words max).
- Branch from `main` (or `develop` if the project uses gitflow).
- One feature/fix per branch — no bundling unrelated work.

## Commit Messages — Conventional Commits

```
{type}({scope}): {short description}

{optional body — explain WHY, not what}

{optional footer: BREAKING CHANGE, Closes #issue}
```

**Types**:

| Type | Use for |
|---|---|
| `feat` | New feature (triggers minor version bump) |
| `fix` | Bug fix (triggers patch version bump) |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Refactoring, no feature or fix |
| `test` | Adding or fixing tests |
| `chore` | Build, tooling, dependencies |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration |
| `revert` | Reverting a commit |

**Scope** — optional, use the module or area: `(auth)`, `(api)`, `(db)`, `(ui)`, `(config)`

**Subject line rules**:
- Imperative mood: "add login form" not "added login form"
- No capital letter at start
- No period at end
- Max 72 characters

**Examples**:
```
feat(auth): add JWT refresh token rotation

fix(api): return 404 instead of 500 when user not found

chore(deps): upgrade TypeScript to 5.4

feat(checkout): add Stripe payment integration

BREAKING CHANGE: payment_method field renamed to payment_source
Closes #89
```

## Pull Requests

**PR Size**:
- Target: under 400 lines changed
- Hard limit: 800 lines. Larger PRs must be split.
- Exception: auto-generated files (migrations, lockfiles) don't count toward the limit

**PR Description must include**:
```markdown
## What
[1–2 sentences describing the change]

## Why
[Business or technical motivation]

## How
[Brief implementation approach, key decisions]

## Testing
[How was this tested? Manual steps or automated tests]

## Screenshots (if UI change)

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated (if applicable)
- [ ] No secrets or credentials committed
- [ ] Self-reviewed the diff
```

**Review Rules**:
- At least 1 approval before merging (2 for changes to auth, payments, or data schema)
- All CI checks must pass
- Resolve all comments before merging (or explicitly note as deferred with a ticket)
- No self-merges on protected branches

## Branch Protection (main / develop)

- Require PR to merge — no direct pushes
- Require CI to pass
- Require at least 1 review
- No force pushes
- Delete branch after merge

## Merge Strategy

- **Squash merge** for feature/fix branches onto main (clean history, one commit per PR)
- **Merge commit** for release branches (preserve release boundary)
- **Rebase** for keeping a long-lived feature branch up to date with main

## Tagging / Releases

Follow semantic versioning: `v{MAJOR}.{MINOR}.{PATCH}`

- `feat` commits → bump MINOR
- `fix` commits → bump PATCH
- `BREAKING CHANGE` footer → bump MAJOR

Tag format: `v1.2.3` (not `1.2.3`)
