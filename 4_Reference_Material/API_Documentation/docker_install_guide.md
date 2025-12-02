# Docker Installation Guide for Ubuntu 24.04 LTS

This guide provides step-by-step instructions for installing Docker Engine on Ubuntu 24.04 LTS (Noble Numbat).

## Prerequisites

- Ubuntu 24.04 LTS system
- User account with sudo privileges
- Internet connection

## Installation Steps

### Step 1: Install Required Packages

Update the package index and install necessary dependencies:

```bash
sudo apt update
sudo apt install apt-transport-https curl
```

**What this does:**
- `apt-transport-https`: Allows APT to retrieve packages over HTTPS
- `curl`: Command-line tool for transferring data with URLs

### Step 2: Add Docker's Official GPG Key

Download and store Docker's GPG key for package verification:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

**What this does:**
- Downloads Docker's official GPG key
- Stores it in `/etc/apt/keyrings/` for repository authentication
- Ensures packages are verified and secure

### Step 3: Set Up Docker's Stable Repository

Add Docker's official repository to your system:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**What this does:**
- Automatically detects your system architecture
- Configures the repository with the correct Ubuntu codename
- Creates the repository file in `/etc/apt/sources.list.d/`

Update the package index with Docker packages:

```bash
sudo apt update
```

### Step 4: Install Docker Engine

Install Docker Engine and its components:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Components installed:**
- **docker-ce**: The Docker Engine itself (Community Edition)
- **docker-ce-cli**: Command-line interface for interacting with Docker daemon
- **containerd.io**: Container runtime that manages container lifecycle
- **docker-buildx-plugin**: Extended build capabilities with support for multi-platform builds
- **docker-compose-plugin**: Tool for defining and running multi-container applications

### Step 5: Verify Docker Service

Check that Docker service is running:

```bash
sudo systemctl is-active docker
```

Expected output: `active`

### Step 6: Verify Installation

Run the test container to confirm Docker is working:

```bash
sudo docker run hello-world
```

**Expected output:**
- Docker will download the hello-world image
- Run a container that prints a confirmation message
- Exit automatically

### Step 7: Configure Non-Root User Access (Recommended)

Allow your user to run Docker commands without sudo:

```bash
sudo usermod -aG docker ${USER}
newgrp docker
```

**What this does:**
- Adds your current user to the `docker` group
- Applies the group membership immediately with `newgrp`

**Note:** You may need to log out and back in for group changes to fully take effect.

### Step 8: Verify Docker Compose

Check Docker Compose installation:

```bash
docker compose version
```

Expected output will show the installed Docker Compose version.

## Post-Installation Verification

Test Docker without sudo:

```bash
docker run hello-world
docker ps -a
docker images
```

## Troubleshooting

### Docker service not starting
```bash
sudo systemctl status docker
sudo journalctl -xeu docker.service
```

### Permission denied errors
Ensure you've logged out and back in after adding user to docker group:
```bash
groups
# Should show 'docker' in the list
```

### Cannot connect to Docker daemon
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

## Useful Docker Commands

- `docker --version` - Display Docker version
- `docker info` - Display system-wide information
- `docker ps` - List running containers
- `docker images` - List downloaded images
- `docker system prune` - Clean up unused containers, networks, and images

## Uninstalling Docker

If you need to remove Docker:

```bash
sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

## Additional Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## References

Based on the official Docker installation guide:
https://linuxiac.com/how-to-install-docker-on-ubuntu-24-04-lts/