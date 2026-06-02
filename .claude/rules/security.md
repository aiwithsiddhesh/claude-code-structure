# Security Standards

These are non-negotiables. Every agent writing code must comply. These rules are not optional under any circumstances.

## Secrets Management

- **NEVER commit secrets, API keys, passwords, tokens, or credentials** to version control. No exceptions.
- Use environment variables for all secrets. Reference via `process.env.SECRET_NAME` (JS) or `os.environ["SECRET_NAME"]` (Python).
- All secret names go in `.env.example` with placeholder values (e.g., `DATABASE_URL=your_database_url_here`).
- Use a secrets manager in production: AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault, or equivalent.
- If a secret is accidentally committed: treat it as compromised immediately. Rotate it, then clean git history.
- `.env` files are always in `.gitignore`. Verify before first commit.

## Input Validation

- **All user input must be validated server-side.** Client-side validation is UX, not security.
- Validate: type, format, length, range, allowed characters/values.
- Reject unknown fields — don't pass `...req.body` directly to database operations.
- Validate file uploads: type (check magic bytes, not just extension), size, filename sanitization.
- Use a validation library: Zod / Joi / Yup (TypeScript), Pydantic (Python). Do not write custom validators for common formats.

## SQL / Database

- **Parameterized queries always.** No string interpolation into SQL. No exceptions.
- ORM query builders are acceptable when they generate parameterized queries under the hood.
- No raw `eval()` or dynamic code execution with user input.
- Verify at code review: if you see a user variable concatenated into a query string, it's a blocker.

## Authentication & Authorization

- Hash passwords with bcrypt, argon2, or scrypt. Minimum cost factor: bcrypt rounds ≥ 12.
- Never log passwords, tokens, or secrets — not even partially.
- JWT tokens: use `RS256` or `ES256` (asymmetric) for services that don't need to issue tokens themselves; `HS256` is acceptable for single-service tokens when the secret is strong.
- JWT expiry: access tokens ≤ 15 minutes. Refresh tokens ≤ 7 days, rotated on use.
- Always verify authorization on the backend — never trust the frontend to enforce access control.
- Check object ownership: verify the requesting user owns the resource, not just that they're authenticated.
- Implement rate limiting on: login, registration, password reset, and any unauthenticated endpoint.

## HTTP & API Security

- **HTTPS only** in all non-local environments.
- Set security headers: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security`.
- CORS: never use `*` in production. Explicitly whitelist allowed origins.
- Don't expose internal error details in API responses. Log internally, return generic messages to clients.
- Never expose stack traces, file paths, or internal IDs in error responses.

## Dependency Security

- Pin dependency versions (use lockfiles: `package-lock.json`, `poetry.lock`, `requirements.txt`).
- Run dependency audit before release: `npm audit` / `pip-audit` / `safety`.
- No dependencies with known Critical or High CVEs unmitigated at release time.
- Review what a new package does before adding it. Prefer established packages over unknown single-purpose ones.

## Error Handling & Logging

- Never log: passwords, tokens, full credit card numbers, SSNs, or other PII.
- Log security events: failed login attempts, permission denials, account lockouts.
- Implement a global error handler — unhandled exceptions must not crash the process silently.
- Return appropriate HTTP status codes: 401 (unauthenticated), 403 (unauthorized), not 500.

## File Operations

- Sanitize all file paths provided by users. Prevent path traversal (`../`).
- Restrict file operations to intended directories only.
- Never execute user-supplied filenames or content.

## Code Review Security Checklist

Before any PR is approved, verify:
- [ ] No secrets or credentials in diff
- [ ] All user inputs validated
- [ ] No SQL string concatenation
- [ ] Auth checks present on all protected routes
- [ ] Dependency changes reviewed
- [ ] Error responses don't leak internals
