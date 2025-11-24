# Fly.io PostgreSQL Dump Restore Guide

This guide shows how to upload and restore a PostgreSQL dump file to your fly.io Postgres database.

## Prerequisites

- flyctl CLI installed and authenticated
- PostgreSQL dump file (`.dump` format)
- Fly.io Postgres database deployed

## Step 1: Upload Dump File to Fly.io

Upload your local dump file to the fly.io Postgres machine:

```bash
flyctl ssh sftp put "path/to/your/myapp.dump" /tmp/myapp.dump --app badeplaner
```

**Example:**
```bash
flyctl ssh sftp put "d:\EMC2\Final Application\my-fullstack-app\myapp.dump" /tmp/myapp.dump --app badeplaner
```

## Step 2: Connect to Postgres Machine

Open an SSH console to the Postgres machine:

```bash
flyctl ssh console --app badeplaner
```

## Step 3: Restore the Database

Once connected to the SSH console, run these commands:

### Option A: Drop and Recreate Database (Recommended for clean restore)

```bash
# Connect to postgres database first
psql -U postgres -d postgres

# Inside psql, drop and recreate the database
DROP DATABASE IF EXISTS myapp;
CREATE DATABASE myapp;

# Exit psql
\q

# Now restore the dump
pg_restore -U postgres -d myapp -v /tmp/myapp.dump
```

### Option B: Restore to Existing Database (Use with caution)

```bash
# Restore without dropping existing data (may cause conflicts)
pg_restore -U postgres -d myapp --clean --if-exists -v /tmp/myapp.dump
```

## Step 4: Verify Restore

After restoration, verify the data was restored correctly:

```bash
# Connect to the database
psql -U postgres -d myapp

# Check tables
\dt

# Check data in a table (replace 'your_table' with actual table name)
SELECT COUNT(*) FROM your_table;

# Exit psql
\q
```

## Alternative: Using psql Console (if you have it open)

If you already have `flyctl postgres connect --app badeplaner` running in another terminal:

1. In a separate terminal, upload the file:
   ```bash
   flyctl ssh sftp put "path/to/myapp.dump" /tmp/myapp.dump --app badeplaner
   ```

2. In the SSH console (from Step 2 above), run the restore commands.

## Troubleshooting

### Permission Issues
If you get permission errors:
```bash
# Make sure the file has correct permissions
chmod 644 /tmp/myapp.dump
```

### Connection Issues
If pg_restore fails to connect:
```bash
# Try with explicit host
pg_restore -U postgres -h localhost -d myapp -v /tmp/myapp.dump
```

### Large Dump Files
For very large dump files, the restore might take time:
```bash
# Monitor progress (if supported by your pg_restore version)
pg_restore -U postgres -d myapp --verbose /tmp/myapp.dump
```

### Clean Restore Issues
If `--clean` doesn't work as expected:
```bash
# Manual cleanup before restore
psql -U postgres -d myapp -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public; GRANT ALL ON SCHEMA public TO postgres; GRANT ALL ON SCHEMA public TO public;"
pg_restore -U postgres -d myapp -v /tmp/myapp.dump
```

## Post-Restore Steps

1. **Update your backend DATABASE_URL** if needed
2. **Test your application** to ensure data is accessible
3. **Run database migrations** if your schema has changed:
   ```bash
   flyctl ssh console --app bathroom-quote-backend --command "npx prisma migrate deploy"
   ```

## File Locations

- **Local dump file**: `d:\EMC2\Final Application\my-fullstack-app\myapp.dump`
- **Remote dump file**: `/tmp/myapp.dump` (on fly.io machine)
- **Database name**: `myapp`
- **Postgres user**: `postgres`

## Security Notes

- The dump file contains your database schema and data
- Store dump files securely and don't commit them to version control
- Consider encrypting sensitive dump files
- Clean up temporary files after restore:
  ```bash
  rm /tmp/myapp.dump
  ```