# Coding Standards

These standards apply to all agents writing or reviewing code in this project.

## Output Directory

All generated project code MUST be saved under `output/{project-name}/`.
- Create the directory before writing any files.
- Project name is derived from the project kickoff or `start-project` skill.
- Example: a project called "customer-portal" → all code in `output/customer-portal/`.
- Never write generated project files to the repo root.

## Naming Conventions

| Context | Convention | Example |
|---|---|---|
| TypeScript/JS variables | camelCase | `userProfile`, `isLoading` |
| TypeScript/JS functions | camelCase | `fetchUserData()`, `handleSubmit()` |
| React components | PascalCase | `UserCard`, `DashboardLayout` |
| TypeScript interfaces/types | PascalCase | `UserProfile`, `ApiResponse<T>` |
| TypeScript enums | PascalCase, UPPER members | `enum Status { ACTIVE, INACTIVE }` |
| CSS classes / Tailwind | kebab-case (custom only) | `user-card__header` |
| Python variables/functions | snake_case | `user_profile`, `fetch_user_data()` |
| Python classes | PascalCase | `UserProfile`, `ApiClient` |
| Python constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| File names (TS/JS) | kebab-case | `user-profile.ts`, `use-auth.ts` |
| File names (React) | PascalCase | `UserCard.tsx`, `DashboardLayout.tsx` |
| File names (Python) | snake_case | `user_service.py`, `auth_utils.py` |
| Database tables | snake_case, plural | `user_profiles`, `order_items` |
| Database columns | snake_case | `created_at`, `user_id` |
| Environment variables | UPPER_SNAKE_CASE | `DATABASE_URL`, `JWT_SECRET` |

## TypeScript / JavaScript

- **Strict mode always on** — `"strict": true` in tsconfig. No exceptions.
- **No `any`** — Use `unknown` and narrow, or define a proper type.
- **No non-null assertions (`!`)** without an explicit comment explaining why it's safe.
- **Explicit return types** on all exported functions.
- **Interfaces over type aliases** for object shapes that will be extended; `type` for unions/intersections.
- **Barrel exports** (`index.ts`) only at module boundaries, not everywhere.
- **Async/await** over raw Promises. No `.then().catch()` chains unless intentional.
- **Imports ordered**: 1) Node built-ins, 2) External packages, 3) Internal absolute, 4) Relative. Blank line between groups.

## Python

- **PEP 8** — enforced via `ruff` or `flake8`.
- **Type hints** on all function signatures.
- **Docstrings** on all public functions, classes, and modules (Google style).
- **`f-strings`** over `.format()` or `%` formatting.
- **Dataclasses or Pydantic models** for structured data, not plain dicts.
- **`pathlib.Path`** over `os.path`.
- **`logging`** module, not `print()`, in production code.

## Code Quality Non-Negotiables

- **Max function length: 40 lines.** Longer functions must be decomposed.
- **Max file length: 300 lines.** Longer files must be split by responsibility.
- **No commented-out code** committed to main. Use git if you need history.
- **No magic numbers** — extract constants with descriptive names.
- **One responsibility per function** — if you need "and" to describe it, split it.
- **Error messages must be actionable** — "Invalid input" is not acceptable. "Email must be a valid address" is.

## Comments

- **Write WHY, not WHAT.** The code shows what; the comment explains the decision.
- **`TODO:`** comments must include a ticket/issue reference: `// TODO(#142): remove after migration`
- **JSDoc/docstrings** required on all exported/public APIs.

## Formatting

- **Indentation**: 2 spaces (TypeScript/JavaScript), 4 spaces (Python).
- **Max line length**: 100 characters.
- **Formatter**: Prettier (TS/JS), Black (Python). Auto-format before commit.
- **Trailing commas**: always in multi-line TypeScript/JavaScript.
