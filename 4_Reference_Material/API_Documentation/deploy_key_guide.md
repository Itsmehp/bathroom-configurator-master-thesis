# Deploy Key Setup Guide

Quick guide for setting up SSH deploy keys and deploying applications from GitHub to your server.

## Prerequisites

- Server with SSH access
- GitHub repository access
- Docker and Docker Compose installed

## Step 1: Generate SSH Deploy Key

Generate an SSH key on your server:

```bash
ssh-keygen -t ed25519 -C "deploy@myserver"
```

**When prompted:**
- Press Enter for default location (`~/.ssh/id_ed25519`)
- Press Enter twice for no passphrase (deploy keys shouldn't have passphrases)

## Step 2: Copy Public Key

Display and copy the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output (starts with `ssh-ed25519`).

## Step 3: Add Deploy Key to GitHub

1. Go to your GitHub repository
2. Navigate to **Settings** → **Deploy keys**
3. Click **Add deploy key**
4. Paste your public key
5. Give it a name (e.g., "Production Server")
6. Check **Allow write access** if needed (usually not required for deployments)
7. Click **Add key**

## Step 4: Test SSH Connection

Verify the key works:

```bash
ssh -T git@github.com
```

**Expected output:**
```
Hi username/repo! You've successfully authenticated, but GitHub does not provide shell access.
```

## Step 5: Clone Repository

Navigate to deployment directory and clone:

```bash
cd /opt
sudo git clone git@github.com:username/repository-name.git
cd repository-name
```

**Example:**
```bash
cd /opt
sudo git clone git@github.com:Itsmehp/bathroom-quote-gen.git
cd bathroom-quote-gen
```

## Step 6: Build Docker Images

Build the Docker images:

```bash
docker compose build
```

This may take several minutes depending on your application.

## Step 7: Start Database First

Start PostgreSQL container first:

```bash
docker compose up postgres -d
```

**Flags:**
- `-d` = detached mode (runs in background)

## Step 8: Restore Database (if needed)

If you have a database dump to restore:

```bash
# Copy dump file to server (from local machine)
scp /path/to/backup.dump user@server:/opt/repository-name/

# Copy dump into container
docker cp backup.dump postgres-container-name:/tmp/backup.dump

# Restore the dump
docker exec -i postgres-container-name pg_restore -U dbuser -d dbname --clean --no-owner -v /tmp/backup.dump
```

## Step 9: Start Remaining Services

Once database is ready, start other services:

```bash
# Start all services
docker compose up -d

# Or start specific services
docker compose up backend -d
docker compose up frontend -d
```

## Step 10: Verify Deployment

Check running containers:

```bash
docker compose ps
```

Check logs:

```bash
# All services
docker compose logs

# Specific service
docker compose logs postgres
docker compose logs backend -f  # -f for follow/live logs
```

## Updating the Deployment

When you need to deploy updates:

```bash
cd /opt/repository-name

# Pull latest changes
sudo git pull

# Rebuild and restart
docker compose down
docker compose build
docker compose up -d
```

## Useful Commands

### Git Operations
```bash
# Check current branch
git branch

# Pull latest changes
git pull origin main

# View commit history
git log --oneline -5
```

### Docker Compose Operations
```bash
# View running containers
docker compose ps

# Stop all services
docker compose down

# Restart a specific service
docker compose restart backend

# View logs
docker compose logs -f

# Remove everything including volumes
docker compose down -v
```

## Troubleshooting

### SSH Key Not Working

Check key permissions:
```bash
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Test with verbose output:
```bash
ssh -vT git@github.com
```

### Permission Denied When Cloning

Use sudo or change ownership:
```bash
sudo chown -R $USER:$USER /opt/repository-name
```

### Container Won't Start

Check logs for errors:
```bash
docker compose logs service-name
```

Check if port is already in use:
```bash
sudo lsof -i :PORT_NUMBER
```

### Database Connection Issues

Verify database is running:
```bash
docker compose ps postgres
```

Check database logs:
```bash
docker compose logs postgres
```

Test connection:
```bash
docker exec -it postgres-container-name psql -U dbuser -d dbname
```

## Security Best Practices

1. **Deploy Keys**: Use read-only deploy keys when possible
2. **Permissions**: Set proper file permissions on SSH keys
3. **Separate Keys**: Use different SSH keys for different servers
4. **Environment Variables**: Store sensitive data in `.env` files (never commit these)
5. **Regular Updates**: Keep your application and dependencies updated

## Quick Reference

```bash
# Complete deployment workflow
cd /opt
sudo git clone git@github.com:username/repo.git
cd repo
docker compose build
docker compose up postgres -d
# (restore database if needed)
docker compose up -d
docker compose ps

# Update workflow
cd /opt/repo
sudo git pull
docker compose down
docker compose build
docker compose up -d
```

---

**Note:** Replace `username/repository-name`, `postgres-container-name`, `dbuser`, and `dbname` with your actual values.