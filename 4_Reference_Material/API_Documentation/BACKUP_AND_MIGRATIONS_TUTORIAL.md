# PostgreSQL Backup, Drift Resolution & Prisma Migration Tutorial

This guide walks through safely backing up Postgres database (running in Docker), resolving Prisma migration drift **without losing data**, and applying a new schema change (example: adding `minEinbau` / `maxEinbau` to `ShowerCabinDetail`).

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Identify the Problem (Drift)](#identify-the-problem-drift)
3. [Create a Safe Backup](#create-a-safe-backup)
   - [Custom Format Dump (Recommended)](#custom-format-dump-recommended)
   - [Globals Dump (Optional)](#globals-dump-optional)
   - [Plain SQL + gzip (Optional)](#plain-sql--gzip-optional)
   - [Volume Snapshot (Alternative)](#volume-snapshot-alternative)
4. [Verify Backup](#verify-backup)
5. [Adopt Existing DB State (Baseline Migration)](#adopt-existing-db-state-baseline-migration)
6. [Apply New Schema Change](#apply-new-schema-change)
7. [Restore (If Ever Needed)](#restore-if-ever-needed)
8. [Troubleshooting](#troubleshooting)
9. [FAQ](#faq)
10. [Reference Commands (Cheat Sheet)](#reference-commands-cheat-sheet)
11. [Fly.io Restore Guide](./postgres_backup_restore_guide.md#flyio-restore-guide)

---

## Prerequisites

- Running Postgres via `docker-compose` (service name: `postgres`).
- Prisma schema at: `backend/prisma/schema.prisma`.
- The `.env` contains a valid `DATABASE_URL` pointing to the container:
  ```env
  DATABASE_URL=postgresql://myuser:mypassword@localhost:5432/myapp?schema=public
  ```
- Node & Prisma CLI installed (available via `npx`).
- PowerShell (Windows) for host commands.

---

## Identify the Problem (Drift)

An error like:

```text
Drift detected: The database schema is not in sync with the migration history.
```

Meaning: The **actual database** contains changes (enums, columns, defaults) that were applied outside the recorded migration files.

Resetting would drop data. Instead, we **adopt** current DB state into a synthetic baseline migration and then continue normally.

---

## Create a Safe Backup

Always back up first.

### Custom Format Dump for Windows (Recommended)

Use this if the postgres docker container is locally installed on Windows

```powershell
$ts = Get-Date -Format "yyyyMMdd_HHmmss"
$container = "my-fullstack-app-postgres-1"  # Replace if different
New-Item -ItemType Directory -Path .\backups -ErrorAction SilentlyContinue | Out-Null
# Create dump inside container
docker exec $container pg_dump -U myuser -d myapp -Fc -f /tmp/myapp_$ts.dump
# Copy to host
docker cp $container:/tmp/myapp_$ts.dump .\backups\
docker cp ${container}:/tmp/myapp_${ts}.dump .\backups\

```
Or
```powershell
docker exec -i my-fullstack-app-postgres-1 pg_dump -U myuser -F c -b -v -f /tmp/myapp041125.dump myapp
```

## Adopt Existing DB State (Baseline Migration)

Goal: Tell Prisma "this is our starting point" matching the live DB, so future migrations apply cleanly.

### 1. (Temporary) Create a Schema Copy Without New Pending Fields

If there are new fields added (e.g., `minEinbau`, `maxEinbau`) **remove or comment them** in a temporary copy:

```powershell
Copy-Item .\backend\prisma\schema.prisma .\backend\prisma\schema.tmp.prisma
# Edit schema.tmp.prisma and remove the new lines that have NOT been applied yet.
```

For Linux / macOS (bash or zsh) users can perform the same steps with the following commands:

```bash
# Copy the schema to a temporary file
cp backend/prisma/schema.prisma backend/prisma/schema.tmp.prisma
# Edit `backend/prisma/schema.tmp.prisma` and remove the new fields that have NOT been applied yet.
```

### 2. Generate Baseline SQL (Empty -> Current Schema)

```powershell
cd backend
npx prisma migrate diff --from-empty --to-schema-datamodel prisma\schema.tmp.prisma --script > baseline.sql
```
Then generate the baseline SQL (same command, use forward slashes on Windows Subsystem for Linux or macOS):
```bash
cd backend
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.tmp.prisma --script > baseline.sql
```
### 3. Create a Migration Folder Manually

Pick a timestamp-like prefix (adjust as needed):

```
backend/prisma/migrations/20250930104500_adopt_existing_state/
  migration.sql
  README.md
```

Move the generated SQL:

```powershell
$folder = "prisma/migrations/20250930104500_adopt_existing_state"
New-Item -ItemType Directory -Path $folder | Out-Null
Move-Item baseline.sql "$folder/migration.sql"
"Adopt existing DB state due to prior drift." | Out-File "$folder/README.md"
```

For Linux/macOS (bash or zsh):

```bash
mkdir -p prisma/migrations/20250930104500_adopt_existing_state
mv baseline.sql prisma/migrations/20250930104500_adopt_existing_state/migration.sql
echo "Adopt existing DB state due to prior drift." > prisma/migrations/20250930104500_adopt_existing_state/README.md
```

### 4. Mark Migration As Applied (Without Running It)

```powershell
npx prisma migrate resolve --applied 20250930104500_adopt_existing_state
```

For Linux/macOS (bash or zsh):

```bash
npx prisma migrate resolve --applied 20250930104500_adopt_existing_state
```

### 5. Clean Up Temp Schema

```powershell
Remove-Item prisma\schema.tmp.prisma
```

For Linux/macOS (bash or zsh):

```bash
rm prisma/schema.tmp.prisma
```

---

## Apply New Schema Change

Now ensure the real `schema.prisma` contains the intended new fields, e.g.:

```prisma
model ShowerCabinDetail {
  // ...existing fields
  minEinbau Float?
  maxEinbau Float?
}
```

Generate the new migration:

```powershell
npx prisma migrate dev --name add_min_max_einbau
```

For Linux/macOS (bash or zsh):

```bash
npx prisma migrate dev --name add_min_max_einbau
```

This should produce a migration adding the two nullable columns only.

(Optional) Regenerate client explicitly:

```powershell
npx prisma generate
```

For Linux/macOS (bash or zsh):

```bash
npx prisma generate
```

Test query:

```ts
const detail = await prisma.showerCabinDetail.findFirst();
console.log(detail?.minEinbau, detail?.maxEinbau);
```

---

## Restore (If Ever Needed)

### Restore From Custom Format Dump

Copy dump back (if not already in container):

```powershell
docker cp .\backups\myapp_YYYYMMDD_HHMMSS.dump my-fullstack-app-postgres-1:/tmp/restore.dump
```

For Linux/macOS (bash or zsh):

```bash
docker cp ./backups/myapp_YYYYMMDD_HHMMSS.dump my-fullstack-app-postgres-1:/tmp/restore.dump
```

Inside container / via exec:

```powershell
docker exec my-fullstack-app-postgres-1 dropdb -U myuser myapp
docker exec my-fullstack-app-postgres-1 createdb -U myuser myapp
docker exec my-fullstack-app-postgres-1 pg_restore -U myuser -d myapp --clean --if-exists /tmp/restore.dump
```

For Linux/macOS (bash or zsh):

```bash
docker exec my-fullstack-app-postgres-1 dropdb -U myuser myapp
docker exec my-fullstack-app-postgres-1 createdb -U myuser myapp
docker exec my-fullstack-app-postgres-1 pg_restore -U myuser -d myapp --clean --if-exists /tmp/restore.dump
```

### Restore Globals (If Used)

```powershell
docker exec -i my-fullstack-app-postgres-1 psql -U myuser -d postgres < .\backups\globals_YYYYMMDD_HHMMSS.sql
```

For Linux/macOS (bash or zsh):

```bash
docker exec -i my-fullstack-app-postgres-1 psql -U myuser -d postgres < ./backups/globals_YYYYMMDD_HHMMSS.sql
```

### Restore From Volume Snapshot

Stop container, create fresh volume, untar into it, restart (must match Postgres major version).

---

## Troubleshooting

| Issue                                | Cause                             | Fix                                         |
| ------------------------------------ | --------------------------------- | ------------------------------------------- |
| Drift still reported                 | Missed a schema difference        | Re-run diff; check enums/defaults alignment |
| `docker cp` wildcard fails           | Docker doesn’t expand `*`         | Use exact filename (`ls /tmp`)              |
| Empty dump file                      | Ran pg_dump inside `psql` prompt  | Exit with `\q` first                        |
| Migration adds unexpected drops      | Temp schema still had new fields  | Remove them before baseline diff            |
| Permission denied on volume snapshot | Container running                 | Stop container first                        |
| Columns recreated in diff            | Type/default/nullability mismatch | Ensure schema copy matches DB exactly       |

---

## FAQ

**Q: Why not just `prisma migrate reset`?**  
Because it drops all data—bad if you have valuable dev/test seed modifications.

**Q: When should I move fields from detail tables to Product?**  
When multiple product categories share the same attributes and you need unified filtering.

**Q: Are custom-format dumps portable?**  
Yes across same major Postgres versions; use plain SQL for maximum portability.

**Q: Can I automate backups?**  
Yes: add a PowerShell script or Git pre-migration hook invoking `pg_dump`.

---

## Reference Commands (Cheat Sheet)

Backup:

```powershell
$ts = Get-Date -Format yyyyMMdd_HHmmss
$container = "my-fullstack-app-postgres-1"
docker exec $container pg_dump -U myuser -d myapp -Fc -f /tmp/myapp_$ts.dump
docker cp $container:/tmp/myapp_$ts.dump .\backups\
```

For Linux/macOS (bash or zsh):

```bash
ts=$(date +%Y%m%d_%H%M%S)
container="my-fullstack-app-postgres-1"
docker exec $container pg_dump -U myuser -d myapp -Fc -f /tmp/myapp_$ts.dump
docker cp $container:/tmp/myapp_$ts.dump ./backups/
```

Baseline diff:

```powershell
npx prisma migrate diff --from-empty --to-schema-datamodel prisma\schema.tmp.prisma --script > baseline.sql
```

For Linux/macOS (bash or zsh):

```bash
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.tmp.prisma --script > baseline.sql
```

Resolve baseline:

```powershell
npx prisma migrate resolve --applied 20250930104500_adopt_existing_state
```

For Linux/macOS (bash or zsh):

```bash
npx prisma migrate resolve --applied 20250930104500_adopt_existing_state
```

New migration:

```powershell
npx prisma migrate dev --name add_min_max_einbau
```

For Linux/macOS (bash or zsh):

```bash
npx prisma migrate dev --name add_min_max_einbau
```

Restore:

```powershell
docker exec $container dropdb -U myuser myapp
docker exec $container createdb -U myuser myapp
docker exec $container pg_restore -U myuser -d myapp --clean --if-exists /tmp/restore.dump
```

For Linux/macOS (bash or zsh):

```bash
docker exec $container dropdb -U myuser myapp
docker exec $container createdb -U myuser myapp
docker exec $container pg_restore -U myuser -d myapp --clean --if-exists /tmp/restore.dump
```
