# Prisma ORM Documentation

**Library:** Prisma
**Library ID:** `/websites/prisma_io`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

Prisma is a next-generation TypeScript ORM that makes database access type-safe and developer-friendly. It provides a declarative schema, automated migrations, and a fully type-safe query builder for Node.js and TypeScript.

## Key Features for Our Project

- **Type-Safe Queries** - Auto-generated TypeScript types from schema
- **PostgreSQL Support** - First-class support for our database
- **Schema-First Approach** - Define models in readable schema file
- **Migration Management** - Version control for database changes
- **Prisma Client** - Intuitive query API with IntelliSense

## Installation

```bash
npm install prisma @prisma/client
npx prisma init
```

This creates:
- `prisma/schema.prisma` - Database schema
- `.env` - Environment variables (DATABASE_URL)

## Schema Definition

### Basic Setup

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  families  FamilyMember[]
}

model Family {
  id        String   @id @default(cuid())
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  members   FamilyMember[]
  events    Event[]
}

model FamilyMember {
  id       String @id @default(cuid())
  role     String

  // Foreign keys
  userId   String
  familyId String

  // Relations
  user     User   @relation(fields: [userId], references: [id])
  family   Family @relation(fields: [familyId], references: [id])

  @@unique([userId, familyId])
}

model Event {
  id          String   @id @default(cuid())
  title       String
  startTime   DateTime
  endTime     DateTime
  description String?
  createdAt   DateTime @default(now())

  // Foreign keys
  familyId    String

  // Relations
  family      Family   @relation(fields: [familyId], references: [id])
}
```

### Field Types

```prisma
model Example {
  // Scalars
  stringField   String
  intField      Int
  floatField    Float
  boolField     Boolean
  dateField     DateTime
  jsonField     Json

  // Modifiers
  optional      String?        // Nullable
  withDefault   String @default("value")
  unique        String @unique
  id            String @id @default(cuid())

  // Auto-managed
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}
```

## Migrations

### Create and Apply Migration

```bash
# Create migration from schema changes
npx prisma migrate dev --name init

# Apply migrations in production
npx prisma migrate deploy

# Reset database (DEV ONLY)
npx prisma migrate reset
```

### Migration Workflow

1. Update `schema.prisma`
2. Run `prisma migrate dev --name descriptive_name`
3. Prisma generates SQL migration file
4. Migration applied to database
5. Prisma Client regenerated

## Prisma Client Usage

### Initialize Client

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### CRUD Operations

```typescript
import { prisma } from '@/lib/prisma'

// CREATE
const user = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe'
  }
})

// READ
const allUsers = await prisma.user.findMany()

const oneUser = await prisma.user.findUnique({
  where: { email: 'user@example.com' }
})

const filteredUsers = await prisma.user.findMany({
  where: {
    email: {
      contains: '@example.com'
    }
  },
  orderBy: {
    createdAt: 'desc'
  },
  take: 10
})

// UPDATE
const updated = await prisma.user.update({
  where: { id: userId },
  data: { name: 'Jane Doe' }
})

// DELETE
const deleted = await prisma.user.delete({
  where: { id: userId }
})
```

### Relations and Includes

```typescript
// Fetch user with related data
const userWithFamilies = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    families: {
      include: {
        family: true
      }
    }
  }
})

// Create with relations
const family = await prisma.family.create({
  data: {
    name: 'Smith Family',
    members: {
      create: [
        {
          role: 'parent',
          user: {
            connect: { id: userId }
          }
        }
      ]
    }
  }
})

// Nested writes
const event = await prisma.event.create({
  data: {
    title: 'Family Dinner',
    startTime: new Date('2026-02-15T18:00:00Z'),
    endTime: new Date('2026-02-15T20:00:00Z'),
    family: {
      connect: { id: familyId }
    }
  }
})
```

### Advanced Queries

```typescript
// Transactions
const [user, family] = await prisma.$transaction([
  prisma.user.create({ data: { email: 'user@example.com' } }),
  prisma.family.create({ data: { name: 'New Family' } })
])

// Raw SQL (when needed)
const result = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE "email" = ${email}
`

// Aggregations
const count = await prisma.user.count()

const stats = await prisma.event.aggregate({
  _count: true,
  _min: { startTime: true },
  _max: { endTime: true }
})

// Group by
const eventsByFamily = await prisma.event.groupBy({
  by: ['familyId'],
  _count: true
})
```

## Implementation for Our Project

### Database Schema Strategy

```prisma
// Core models for Phase 2A
model User {
  id            String         @id @default(cuid())
  clerkId       String         @unique  // Clerk auth ID
  email         String         @unique
  name          String?
  timezone      String         @default("America/New_York")
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt

  families      FamilyMember[]
  preferences   UserPreference?
}

model Family {
  id              String          @id @default(cuid())
  name            String
  timezone        String          @default("America/New_York")
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  members         FamilyMember[]
  events          Event[]
  subscriptionId  String?         @unique
  subscription    Subscription?
}

model FamilyMember {
  id        String   @id @default(cuid())
  role      String   // 'admin', 'member'
  userId    String
  familyId  String
  joinedAt  DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  family    Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@unique([userId, familyId])
  @@index([userId])
  @@index([familyId])
}

model Event {
  id           String   @id @default(cuid())
  title        String
  description  String?
  startTime    DateTime
  endTime      DateTime
  timezone     String
  familyId     String
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  family       Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([familyId])
  @@index([startTime])
}

model Subscription {
  id              String   @id @default(cuid())
  familyId        String   @unique
  stripeCustomerId String  @unique
  stripePriceId   String
  status          String   // 'active', 'canceled', 'past_due'
  currentPeriodEnd DateTime
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  family          Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)
}
```

### Common Queries for Our App

```typescript
// Get user's families
export async function getUserFamilies(userId: string) {
  return prisma.familyMember.findMany({
    where: { userId },
    include: {
      family: {
        include: {
          members: {
            include: {
              user: true
            }
          }
        }
      }
    }
  })
}

// Get family events
export async function getFamilyEvents(familyId: string, startDate: Date, endDate: Date) {
  return prisma.event.findMany({
    where: {
      familyId,
      startTime: {
        gte: startDate,
        lte: endDate
      }
    },
    orderBy: {
      startTime: 'asc'
    }
  })
}

// Check user family access
export async function hasAccessToFamily(userId: string, familyId: string) {
  const member = await prisma.familyMember.findUnique({
    where: {
      userId_familyId: {
        userId,
        familyId
      }
    }
  })
  return member !== null
}
```

## Best Practices for Our App

1. **Singleton Pattern** - Use single Prisma Client instance (see lib/prisma.ts)
2. **Connection Pooling** - Prisma handles this automatically
3. **Type Safety** - Never use `any`, leverage generated types
4. **Indexes** - Add `@@index` for frequently queried fields
5. **Cascade Deletes** - Use `onDelete: Cascade` for cleanup
6. **Transactions** - Use for multi-step operations that must succeed together
7. **Environment Variables** - Keep DATABASE_URL in .env, never commit

## Error Handling

```typescript
import { Prisma } from '@prisma/client'

try {
  const user = await prisma.user.create({
    data: { email: 'duplicate@example.com' }
  })
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2002') {
      // Unique constraint violation
      console.error('Email already exists')
    }
  }
  throw error
}
```

## Common Error Codes

- `P2002` - Unique constraint violation
- `P2025` - Record not found
- `P2003` - Foreign key constraint failed
- `P2014` - Relation violation

## Development Workflow

1. Update schema in `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name change_description`
3. Prisma Client auto-regenerates with new types
4. Use new types in application code

## Prisma Studio

```bash
# Open visual database browser
npx prisma studio
```

Access at `http://localhost:5555` to view and edit data visually.

## Environment Variables

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/dbname?schema=public"
```

For production (Vercel + Supabase):
```bash
DATABASE_URL="postgresql://user:password@db.supabase.co:5432/postgres?pgbouncer=true"
DIRECT_URL="postgresql://user:password@db.supabase.co:5432/postgres"
```

## Resources

- Official Docs: https://www.prisma.io/docs
- Schema Reference: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference
- Client API: https://www.prisma.io/docs/reference/api-reference/prisma-client-reference
- PostgreSQL Guide: https://www.prisma.io/docs/guides/database/postgresql
- Best Practices: https://www.prisma.io/docs/guides/performance-and-optimization
