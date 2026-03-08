---
name: db-migrate
description: Helper for Prisma database migrations
tags: [database, prisma, migration]
---

# Database Migration Helper

This skill helps you create and manage Prisma database migrations safely.

## What it does

1. Checks current migration status
2. Shows pending migrations
3. Helps create new migrations
4. Runs migrations safely
5. Handles rollbacks if needed

## Usage

```bash
/db-migrate [action] [name]
```

**Actions:**
- `status` - Check migration status
- `create <name>` - Create new migration
- `run` - Run pending migrations
- `reset` - Reset database (dev only!)
- `seed` - Seed database with initial data

## Commands

### Check Status

```bash
npx prisma migrate status
```

Shows:
- Current database schema version
- Pending migrations
- Migration history

### Create Migration

```bash
npx prisma migrate dev --name <migration_name>
```

**Naming conventions:**
- `create_users_table`
- `add_voice_fields_to_transactions`
- `create_budgets_table`
- `add_indexes_to_transactions`

### Run Migrations

```bash
# Development
npx prisma migrate dev

# Production
npx prisma migrate deploy
```

### Reset Database (Dev Only)

```bash
npx prisma migrate reset
```

**⚠️ WARNING:** This will:
1. Drop all tables
2. Recreate schema
3. Run all migrations
4. Run seed script

### Seed Database

```bash
npx prisma db seed
```

## Safety Checks

Before running migrations:

1. **Backup database** (production)
   ```bash
   pg_dump finnote_prod > backup_$(date +%Y%m%d_%H%M%S).sql
   ```

2. **Review migration SQL**
   ```bash
   cat prisma/migrations/<timestamp>_<name>/migration.sql
   ```

3. **Test on staging first**

4. **Check for data loss**
   - Dropping columns?
   - Changing data types?
   - Removing tables?

## Common Migration Patterns

### Add a new table

```bash
/db-migrate create create_budgets_table
```

Edit `schema.prisma`:
```prisma
model Budget {
  id String @id @default(uuid())
  // ... fields
}
```

Generate migration:
```bash
npx prisma migrate dev --name create_budgets_table
```

### Add a column

```bash
/db-migrate create add_voice_audio_url_to_transactions
```

Edit `schema.prisma`:
```prisma
model Transaction {
  // ... existing fields
  voiceAudioUrl String? @map("voice_audio_url")
}
```

### Add an index

```bash
/db-migrate create add_index_to_transaction_date
```

Edit `schema.prisma`:
```prisma
model Transaction {
  // ... fields

  @@index([transactionDate(sort: Desc)])
}
```

### Rename a column

**⚠️ Careful:** This can cause data loss!

Option 1: Using `@map()` (no migration needed)
```prisma
model Transaction {
  voiceAudioUrl String? @map("voice_audio_url")
}
```

Option 2: Actual rename (requires migration)
```sql
-- In migration.sql
ALTER TABLE transactions
RENAME COLUMN old_name TO new_name;
```

## Seed Data

Create `prisma/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create system categories
  const categories = [
    { name: 'Food', nameVi: 'Ăn uống', icon: '🍔', color: '#FF6B6B', isSystem: true },
    { name: 'Transport', nameVi: 'Di chuyển', icon: '🚗', color: '#4ECDC4', isSystem: true },
    { name: 'Shopping', nameVi: 'Mua sắm', icon: '🛍️', color: '#45B7D1', isSystem: true },
    // ... more categories
  ];

  for (const category of categories) {
    await prisma.category.upsert({
      where: { name: category.name },
      update: {},
      create: category,
    });
  }

  console.log('✅ Seed data created');
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Add to `package.json`:
```json
{
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

## Troubleshooting

### Migration failed

```bash
# Mark migration as rolled back
npx prisma migrate resolve --rolled-back <migration_name>

# Fix the issue in schema.prisma

# Create new migration
npx prisma migrate dev
```

### Schema out of sync

```bash
# Reset Prisma client
npx prisma generate

# Or reset entire database (dev)
npx prisma migrate reset
```

### Cannot connect to database

```bash
# Check if PostgreSQL is running
docker ps | grep postgres

# Start if not running
docker-compose up -d postgres

# Test connection
psql -h localhost -U postgres -d finnote_dev
```

## Migration Checklist

Before deploying migrations to production:

- [ ] Backup database
- [ ] Test on staging environment
- [ ] Review migration SQL for breaking changes
- [ ] Check for potential data loss
- [ ] Verify indexes are created
- [ ] Test rollback procedure
- [ ] Update application code if schema changed
- [ ] Notify team of maintenance window
- [ ] Monitor application after deployment

## Example Workflow

```bash
# 1. Check current status
/db-migrate status

# 2. Create new migration
/db-migrate create add_voice_processing_logs

# 3. Edit schema.prisma
# Add new model or fields

# 4. Generate migration
npx prisma migrate dev --name add_voice_processing_logs

# 5. Review generated SQL
cat prisma/migrations/*/migration.sql

# 6. Test locally
npm run start:dev

# 7. Commit migration files
git add prisma/
git commit -m "feat: Add voice processing logs table"

# 8. Deploy to staging
git push origin develop

# 9. Run migration on staging
npx prisma migrate deploy

# 10. Test on staging
# Verify everything works

# 11. Deploy to production
git push origin main
npx prisma migrate deploy
```

---

*Always test migrations on staging before production!*
