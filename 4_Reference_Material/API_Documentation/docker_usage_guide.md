# Docker Usage Guide for Fullstack App

This guide provides quick commands to build, start, and stop application using Docker Compose.

## Prerequisites

- Docker and Docker Compose installed (see `docker_install_guide.md`)
- Project files in place

## Commands

### Build the Application

Build all services defined in `docker-compose.yml`:

```bash
docker-compose build
```

**What this does:**
- Builds Docker images for backend, frontend, and shared services
- Downloads and installs dependencies as specified in Dockerfiles

### Start the Application

Start all services in detached mode (background):

```bash
docker-compose up -d
```

**What this does:**
- Starts containers for all services
- Runs in background so you can continue using the terminal
- Services will be accessible at their configured ports

### Check Running Services

View status of running containers:

```bash
docker-compose ps

docker ps
```

**What this does:**
- Lists all containers and their current state
- Shows ports and container names

### View Logs

Monitor application logs:

```bash
docker-compose logs -f
```

**What this does:**
- Displays logs from all services
- `-f` follows new log entries in real-time

### Stop the Application

Stop all running services:

```bash
docker-compose down
```

**What this does:**
- Stops and removes all containers
- Preserves data volumes (if configured)

### Stop and Remove Everything

Stop services and remove containers, networks, and volumes:

```bash
docker-compose down -v
```

**What this does:**
- Completely cleans up the Docker environment
- Removes data volumes (use with caution)

## Useful Commands

- **Restart services:** `docker-compose restart`
- **Rebuild and restart:** `docker-compose up -d --build`
- **View specific service logs:** `docker-compose logs backend`
- **Execute commands in container:** `docker-compose exec backend bash`

## Troubleshooting

### Services not starting
```bash
docker-compose logs
# Check for error messages
```

### Port conflicts
```bash
docker-compose ps
# Check which ports are in use
```

### Clean rebuild
```bash
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```

## Services Overview

- **Backend:** API server (typically port 8000)
- **Frontend:** Web application (typically port 3000)
- **Shared:** Common utilities and dependencies

Access the application at `http://localhost:3000` after starting.</content>
<parameter name="filePath">d:\EMC2\Final Application\my-fullstack-app\DOCKER_USAGE_GUIDE.md