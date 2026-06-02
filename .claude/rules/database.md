---
paths:
  - "**/migrations/**"
  - "**/models/**"
  - "**/schemas/**"
  - "prisma/**"
  - "**/database/**"
  - "db/**"
  - "output/**/migrations/**"
  - "output/**/models/**"
  - "output/**/schemas/**"
  - "output/**/prisma/**"
  - "output/**/database/**"
  - "output/**/db/**"
---

# Database Patterns

Loaded when working with migrations, models, schemas, or database configuration.

## Migration Rules

**Naming**: `{timestamp}_{action}_{object}.{ext}`
```
20240615_create_users.sql
20240616_add_email_index_to_users.sql
20240617_alter_orders_add_status_column.sql
```

- Timestamp format: `YYYYMMDD` (or `YYYYMMDDHHmmss` for same-day migrations)
- Action verbs: `create`, `alter`, `drop`, `add`, `remove`, `rename`, `backfill`
- Object: table name, or `{column}_to_{table}` for column-level changes

**Every migration must have a rollback**:
```sql
-- up
ALTER TABLE users ADD COLUMN display_name VARCHAR(255);

-- down
ALTER TABLE users DROP COLUMN display_name;
```

- Test the rollback in staging before merging.
- Migrations that cannot be safely rolled back (irreversible data changes) must be explicitly flagged in PR description with a recovery plan.

**Migration safety**:
- Never modify existing migration files once merged. Write a new one.
- Never drop a column that application code still references — deprecate first, remove in a later migration after code is deployed.
- Adding NOT NULL columns to existing tables requires a default value or a two-step migration (add nullable → backfill → add constraint).
- Large table migrations (> 1M rows) must be batched. Never a single `UPDATE all rows` in production.

## Schema Conventions

**Tables**:
- Plural, snake_case: `users`, `order_items`, `product_categories`
- Every table has: `id` (primary key), `created_at`, `updated_at`
- Soft delete pattern (where used): `deleted_at TIMESTAMP NULL` — never physically delete rows with audit requirements

**Columns**:
- snake_case: `first_name`, `stripe_customer_id`, `is_active`
- Boolean columns: prefix with `is_` or `has_`: `is_active`, `has_verified_email`
- Foreign keys: `{referenced_table_singular}_id`: `user_id`, `order_id`
- Timestamp columns: `_at` suffix: `created_at`, `published_at`, `deleted_at`

**Primary Keys**:
- Default: `UUID` (v4) for distributed systems, application-generated
- `BIGINT SERIAL` acceptable for internal, high-insert tables where UUID overhead matters
- Never use business identifiers (email, phone) as primary keys

**Indexes**:
- Add index for every foreign key column (most ORMs don't do this automatically)
- Add index for every column used in `WHERE`, `ORDER BY`, or `JOIN` in common queries
- Composite index column order matters: most selective column first
- Name indexes explicitly: `idx_{table}_{column(s)}`: `idx_orders_user_id`, `idx_users_email`
- Partial indexes for filtered queries: `CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'pending'`

## Query Patterns

**Parameterized queries always** — see `security.md`. No exceptions.

**N+1 prevention**:
- Identify and eliminate N+1 queries in code review
- Use eager loading / `include` in ORMs for known relationship fetches
- Use `DataLoader` pattern for GraphQL resolvers

**Transaction boundaries**:
- Wrap multi-step operations that must succeed or fail together in a transaction
- Keep transactions short — don't make external API calls inside a transaction
- Explicitly set isolation level when it matters (`SERIALIZABLE` for financial operations)

**Soft deletes**:
- Filter `deleted_at IS NULL` in all queries that should exclude deleted records
- Add a database-level partial index to enforce uniqueness only on non-deleted rows if needed

## ORM Conventions (Prisma)

```prisma
model User {
  id        String   @id @default(uuid())
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  email     String   @unique
  // ...

  @@map("users")  // maps to snake_case table
}
```

- Use `@map` and `@@map` to keep Prisma model names PascalCase while database names stay snake_case.
- Always run `prisma generate` after schema changes.
- Use `prisma migrate dev` in development; `prisma migrate deploy` in CI/production.
- Never use `prisma db push` in production.
