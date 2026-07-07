---
name: prisma-patterns
description: Prisma ORM patterns for TypeScript backends — schema design, query optimization, transactions, pagination, and critical traps like updateMany returning count not records, $transaction timeouts, migrate dev resetting the DB, @updatedAt skipped on bulk writes, and serverless connection exhaustion. Covers Prisma 5/6 and the Prisma 7 breaking changes (driver adapters, prisma.config.ts, generated client output, ESM).
---

# Prisma Patterns

Production patterns and non-obvious traps for Prisma ORM in TypeScript backends.
Tested against Prisma 5.x, 6.x, and 7.x. Some behaviors differ from Prisma 4.

**Prisma 7 is a breaking release** — the generator, client import path, client
instantiation, and datasource/connection configuration all changed. The query, transaction,
pagination, and anti-pattern guidance below applies to all of 5/6/7; the setup examples
(PrismaClient singleton, error import, serverless connection string) are written for 5/6.
Read the **Prisma 7 (breaking changes)** section first if you are on v7, and follow its
signposts on the affected 5/6 examples.

Check the Prisma version before applying version-specific patterns:

```bash
npx prisma --version
```

Prisma 5 introduced `relationJoins`, which can load relations via JOIN rather than separate queries depending on query strategy and configuration. The `omit` field modifier and `prisma.$extends` Client Extensions API were also added. Note: `relationJoins` can cause row explosion on large 1:N relations or deep nested `include` — benchmark both approaches when relations may return many rows per parent.

## Prisma 7 (breaking changes)

Prisma 7 is not a drop-in upgrade. Following the 5/6 setup examples on a v7 project produces
code that will not compile or connect. Requires Node >= 20.19 and TypeScript >= 5.4.
Full guide: https://www.prisma.io/docs/orm/more/upgrade-guides/upgrading-versions/upgrading-to-prisma-7

**MongoDB is not yet supported on Prisma 7** — stay on Prisma 6 for MongoDB.

### Generator and client output

The provider changed from `prisma-client-js` to `prisma-client`, and `output` is now
mandatory — the client is no longer emitted into `node_modules`.

```prisma
generator client {
  provider = "prisma-client"   // was: "prisma-client-js"
  output   = "./generated/prisma"
}
```

### Import path

Import from the generated `output` directory, not `@prisma/client`. This applies to both
`PrismaClient` and the `Prisma` namespace (used for error classes — see Error Handling).

```ts
// v7
import { PrismaClient, Prisma } from './generated/prisma/client';

// v5/6
import { PrismaClient, Prisma } from '@prisma/client';
```

### Driver adapter is now mandatory

Every database connection requires an explicit driver adapter. The connection string is
passed to the adapter, not to `PrismaClient`.

```ts
// lib/prisma.ts (v7 — PostgreSQL)
import { PrismaClient } from './generated/prisma/client';
import { PrismaPg } from '@prisma/adapter-pg';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === 'development' ? ['query', 'error'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

Use the adapter matching your database (`@prisma/adapter-pg`, `@prisma/adapter-mariadb`, etc.).

### `prisma.config.ts` replaces datasource URL config

`url`, `directUrl`, and `shadowDatabaseUrl` move out of the schema `datasource` block into a
`prisma.config.ts` at the project root. The CLI reads connection details from this file;
environment variables are no longer loaded automatically, so load them explicitly.

```ts
// prisma.config.ts
import 'dotenv/config';
import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  migrations: {
    path: 'prisma/migrations',
    seed: 'tsx prisma/seed.ts',
  },
  datasource: {
    url: env('DATABASE_URL'),
    shadowDatabaseUrl: env('SHADOW_DATABASE_URL'),
  },
});
```

### ESM only

Prisma 7 ships as ES modules. Set `"type": "module"` in `package.json`, and in `tsconfig.json`
use `"module": "ESNext"` with `"target": "ES2023"`.

### CLI no longer auto-generates or auto-seeds

- `migrate dev` and `db push` no longer run `prisma generate` — run it explicitly after
  schema changes.
- Migrations no longer run seed scripts automatically — seed manually with `prisma db seed`.
- Removed flags: `--skip-generate` and `--skip-seed` (from `migrate dev` / `db push`);
  `--schema` and `--url` (from `db execute` — connection comes from `prisma.config.ts`).
- `migrate diff`: `--shadow-database-url` is removed (configure it in `prisma.config.ts`);
  the URL-based `--from-url`/`--to-url` become `--from-config-datasource`/`--to-config-datasource`.

### Middleware removed

`$use` middleware is gone. Use Client Extensions (`$extends`) instead. This reinforces the
soft-delete guidance below: filter explicitly rather than hiding behavior in middleware.

## When to Activate

- Designing or modifying Prisma schema models and relations
- Writing queries, transactions, or pagination logic
- Using `updateMany`, `deleteMany`, or any bulk operation
- Running or planning database migrations
- Deploying to serverless environments (Vercel, Lambda, Cloudflare Workers)
- Implementing soft delete or multi-tenant row filtering

## Core Concepts

### ID Strategy

| Strategy | Use When | Avoid When |
|---|---|---|
| `@default(cuid())` | Default choice — URL-safe, sortable, no collisions | Sequential IDs needed for external systems |
| `@default(uuid())` | Interoperability with non-Prisma systems required | High-write tables (random UUIDs fragment B-tree indexes) |
| `@default(autoincrement())` | Internal join tables, audit logs | Public-facing IDs (exposes record count) |

### Schema Defaults

```prisma
model User {
  id        String    @id @default(cuid())
  email     String    @unique  // @unique already creates an index — no @@index needed
  name      String
  role      Role      @default(USER)
  posts     Post[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  @@index([createdAt])
  @@index([deletedAt, createdAt]) // composite for soft-delete + sort queries
}
```

- Add `@@index` on every foreign key and column used in `WHERE` or `ORDER BY`.
- Declare `deletedAt DateTime?` upfront when soft delete is a foreseeable requirement — adding it later requires a migration on a live table.
- `updatedAt @updatedAt` is set automatically by Prisma on `update` and `upsert` only (see Anti-Patterns for bulk update trap).

### `include` vs `select`

| | `include` | `select` |
|---|---|---|
| Returns | All scalar fields + specified relations | Only specified fields |
| Use when | You need most fields plus a relation | Hot paths, large tables, avoiding over-fetch |
| Performance | May over-fetch on wide tables | Minimal payload, faster on large datasets |
| Prisma 5 note | Uses JOIN by default (`relationJoins`) | Same |

```ts
// include — all columns + relation
const user = await prisma.user.findUnique({
  where: { id },
  include: { posts: { select: { id: true, title: true } } },
});

// select — explicit allowlist
const user = await prisma.user.findUnique({
  where: { id },
  select: { id: true, email: true, name: true },
});
```

Never return raw Prisma entities from API responses — map to response DTOs to control exposed fields:

```ts
// BAD: leaks passwordHash, deletedAt, internal fields
return await prisma.user.findUniqueOrThrow({ where: { id } });

// GOOD: explicit DTO mapping
const user = await prisma.user.findUniqueOrThrow({ where: { id } });
return { id: user.id, name: user.name, email: user.email };
```

### Transaction Form Selection

| Situation | Use |
|---|---|
| Independent operations, no inter-dependency | Array form |
| Later step depends on earlier result | Interactive form |
| External calls (email, HTTP) involved | Outside transaction entirely |

```ts
// Array form — batched in one round trip
const [user, post] = await prisma.$transaction([
  prisma.user.update({ where: { id }, data: { name } }),
  prisma.post.create({ data: { title, authorId: id } }),
]);

// Interactive form — use tx client only, never the outer prisma client
const post = await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUniqueOrThrow({ where: { id } });
  if (user.role !== 'ADMIN') throw new Error('Forbidden');
  return tx.post.create({ data: { title, authorId: user.id } });
});
```

### PrismaClient Singleton

Each `PrismaClient` instance opens its own connection pool. Instantiate once.

> Prisma 7: the import path and instantiation differ — the client comes from the generated
> `output` directory and requires a driver adapter. See the Prisma 7 section.

```ts
// lib/prisma.ts (Prisma 5/6)
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

The `globalThis` pattern prevents duplicate instances during hot reload (Next.js, nodemon, ts-node-dev).

### N+1 Problem

Loading relations inside a loop issues one query per row.

```ts
// BAD: N+1 — one extra query per user
const users = await prisma.user.findMany();
for (const user of users) {
  const posts = await prisma.post.findMany({ where: { authorId: user.id } });
}

// GOOD: single query
const users = await prisma.user.findMany({ include: { posts: true } });
```

With Prisma 5+ `relationJoins`, the `include` form uses a single JOIN. On large 1:N sets this may increase result set size — benchmark both approaches if the relation can return many rows per parent.

## Code Examples

### Cursor Pagination (preferred for feeds and large datasets)

```ts
async function getPosts(cursor?: string, limit = 20) {
  const items = await prisma.post.findMany({
    where: { published: true },
    orderBy: [
      { createdAt: 'desc' },
      { id: 'desc' }, // secondary sort prevents unstable pagination on duplicate timestamps
    ],
    take: limit + 1,
    ...(cursor && { cursor: { id: cursor }, skip: 1 }),
  });

  const hasNextPage = items.length > limit;
  if (hasNextPage) items.pop();

  return { items, nextCursor: hasNextPage ? items[items.length - 1].id : null };
}
```

Fetch `limit + 1` and pop — canonical way to detect `hasNextPage` without an extra count query. Always include a unique field (e.g. `id`) as a secondary `orderBy` to prevent unstable pagination when multiple rows share the same timestamp. Use offset pagination only when users need to jump to arbitrary pages (admin tables).

### Soft Delete

```ts
// Always filter explicitly — do not rely on middleware (hides behavior, hard to debug)
const activeUsers = await prisma.user.findMany({ where: { deletedAt: null } });

await prisma.user.update({ where: { id }, data: { deletedAt: new Date() } });
await prisma.user.update({ where: { id }, data: { deletedAt: null } }); // restore
```

### Error Handling

> Prisma 7: import `Prisma` from the generated `output` directory, not `@prisma/client`.
> See the Prisma 7 section. The error codes and handling below are unchanged.

```ts
import { Prisma } from '@prisma/client';

try {
  await prisma.user.create({ data: { email } });
} catch (e) {
  if (e instanceof Prisma.PrismaClientKnownRequestError) {
    if (e.code === 'P2002') throw new ConflictError('Email already exists');
    if (e.code === 'P2025') throw new NotFoundError('Record not found');
    if (e.code === 'P2003') throw new BadRequestError('Referenced record does not exist');
  }
  throw e;
}
```

Common codes: `P2002` unique violation · `P2025` not found · `P2003` foreign key violation.

Catch at the service boundary and translate to domain errors. Never expose raw Prisma messages to API consumers.

### Connection Pool — Serverless

> Prisma 7: the datasource URL is configured in `prisma.config.ts`, not read from the schema
> `datasource` block; connection params can still be embedded in the URL string as shown here.
> See the Prisma 7 section.

Embed connection params directly in `DATABASE_URL` — string concatenation breaks if the URL already has query parameters (e.g. `?schema=public`):

```bash
# .env — preferred: embed params in the URL
DATABASE_URL="postgresql://user:pass@host/db?connection_limit=1&pool_timeout=20"

# With an external pooler (PgBouncer, Supabase pooler)
DATABASE_URL="postgresql://user:pass@host/db?pgbouncer=true&connection_limit=1"
```

```ts
// Vercel, AWS Lambda, and similar serverless runtimes: cap pool to 1 per instance
// connection_limit and pool_timeout are controlled via DATABASE_URL
const prisma = new PrismaClient();
```

## Anti-Patterns

### `updateMany` returns a count, not records

```ts
// BAD: result is { count: 2 } — users[0] is undefined
const users = await prisma.user.updateMany({ where: { role: 'GUEST' }, data: { role: 'USER' } });

// GOOD: capture IDs first, then update, then fetch only the affected rows
const targets = await prisma.user.findMany({
  where: { role: 'GUEST' },
  select: { id: true },
});
const ids = targets.map((u) => u.id);
await prisma.user.updateMany({ where: { id: { in: ids } }, data: { role: 'USER' } });
const updated = await prisma.user.findMany({ where: { id: { in: ids } } });
```

Same applies to `deleteMany` — returns `{ count: n }`, never the deleted rows.

### `$transaction` interactive form times out after 5 seconds

```ts
// BAD: external call inside transaction exceeds 5s default → "Transaction already closed"
await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUniqueOrThrow({ where: { id } });
  await sendWelcomeEmail(user.email); // external call
  await tx.user.update({ where: { id }, data: { emailSent: true } });
});

// GOOD: external calls outside the transaction
const user = await prisma.user.findUniqueOrThrow({ where: { id } });
await sendWelcomeEmail(user.email);
await prisma.user.update({ where: { id }, data: { emailSent: true } });

// Only raise timeout when bulk processing genuinely needs it
await prisma.$transaction(async (tx) => { ... }, { timeout: 30_000 });
```

### `migrate dev` can reset the database

`migrate dev` detects schema drift and may prompt to reset the DB, dropping all data.

```bash
# NEVER on shared dev, staging, or production
npx prisma migrate dev --name add_column

# Safe everywhere except local solo dev
npx prisma migrate deploy

# Check drift without applying
npx prisma migrate diff \
  --from-migrations ./prisma/migrations \
  --to-schema-datamodel ./prisma/schema.prisma \
  --shadow-database-url "$SHADOW_DATABASE_URL"
```

### Manually editing a migration file breaks future deploys

Prisma checksums every migration file. Editing after apply causes `P3006 checksum mismatch` on every environment where the original already ran. Create a new migration instead.

### Breaking schema changes require multi-step migration

Adding `NOT NULL` to an existing column or renaming a column in one migration will lock the table or drop data. Use expand-and-contract (see `database-migrations` for the full cross-ORM zero-downtime strategy):

```bash
# Step 1: create migration locally, then deploy
npx prisma migrate dev --name add_new_column   # local only
npx prisma migrate deploy                       # staging / production
```

```ts
// Step 2: backfill data (run in a script or migration job, not in the shell)
await prisma.user.updateMany({ data: { newColumn: derivedValue } });
```

```bash
# Step 3: create the NOT NULL constraint migration locally, then deploy
npx prisma migrate dev --name make_new_column_required  # local only
npx prisma migrate deploy                               # staging / production
```

### `@updatedAt` does not fire on `updateMany`

`@updatedAt` is set automatically only on `update` and `upsert`. Bulk writes leave it stale.

```ts
// BAD: updatedAt stays at its old value
await prisma.post.updateMany({ where: { authorId }, data: { published: true } });

// GOOD
await prisma.post.updateMany({
  where: { authorId },
  data: { published: true, updatedAt: new Date() },
});
```

### Soft delete + `findUniqueOrThrow` leaks deleted records

`findUniqueOrThrow` throws `P2025` only when the row does not exist in the DB. Soft-deleted rows still exist and are returned without error.

`findUniqueOrThrow` requires a unique constraint field in `where` — adding `deletedAt: null` alongside `id` breaks the type because `{ id, deletedAt }` is not a compound unique constraint. Use `findFirstOrThrow` instead.

```ts
// BAD: returns soft-deleted user
const user = await prisma.user.findUniqueOrThrow({ where: { id } });

// BAD: Prisma type error — { id, deletedAt } is not a unique constraint
const user = await prisma.user.findUniqueOrThrow({ where: { id, deletedAt: null } });

// GOOD: findFirstOrThrow supports arbitrary where conditions
const user = await prisma.user.findFirstOrThrow({ where: { id, deletedAt: null } });
```

### `deleteMany` without `where` deletes every row

```ts
// BAD: silently wipes the table
await prisma.post.deleteMany();

// GOOD
await prisma.post.deleteMany({ where: { authorId: userId } });
```

## Best Practices

| Rule | Reason |
|---|---|
| `migrate deploy` in CI/CD, `migrate dev` only locally | `migrate dev` can reset the DB on drift |
| Map entities to response DTOs | Prevents leaking internal fields |
| Catch `PrismaClientKnownRequestError` at service boundary | Translate to domain errors |
| Prefer `*OrThrow` methods over manual null checks | Throws P2025 automatically; use `findFirstOrThrow` when filtering non-unique fields |
| `connection_limit=1` + external pooler in serverless | Prevents connection exhaustion |
| Always provide `where` on `deleteMany` | Prevents accidental table wipe |
| Set `updatedAt: new Date()` manually in `updateMany` | `@updatedAt` skips bulk writes |

## Related Skills

- `nestjs-patterns` — NestJS service layer that integrates Prisma
- `postgres-patterns` — PostgreSQL-level indexing and connection tuning
- `database-migrations` — multi-step migration planning for production
- `backend-patterns` — general API and service layer design
