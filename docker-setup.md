# Agent Docker Workflow: Per-Project Containers

Standardized agent-flywheel workflow using Docker containers for project-specific environments. Agents run via `docker exec` into dedicated project containers, providing isolated, reproducible environments with full access to project code, dependencies, and credentials.

## Overview

**The Method:** Each project gets its own Docker container with project-specific dependencies. Agents execute commands inside the container via `docker exec`, configured through per-project NTM settings.

**Benefits:**
- **Isolation**: Each project has its own environment and dependencies
- **Reproducibility**: Consistent environments across team members and machines
- **Flexibility**: Project-specific language runtimes, databases, and tools
- **Security**: Containerized execution with credential mounting
- **NTM Integration**: Seamless multi-agent orchestration within containers

## Quick Start

1. **Create Project Dockerfile**
```dockerfile
FROM ruby:latest

# Install base dependencies and agents
RUN apt-get update && apt-get install -y \
    curl git build-essential && \
    rm -rf /var/lib/apt/lists/*

# Install agents
RUN curl -fsSL https://claude.ai/install.sh | sh
RUN curl -fsSL https://bun.sh/install | bash && \
    ~/.bun/bin/bun install -g @openai/codex @google/gemini-cli

# Add project dependencies here
# RUN apt-get install -y python3 nodejs ...

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

2. **Build and Start Container**
```shell
docker build -t myproject-env .
docker run -d --name myproject-env \
  -v "$(pwd):/workspace" \
  -v ~/.claude:/root/.claude \
  -v ~/.codex:/root/.codex \
  -v ~/.config/gemini:/root/.config/gemini \
  myproject-env
```

3. **Configure NTM for Container Usage**
```shell
mkdir -p .ntm
cat > .ntm/config.toml << 'EOF'
[agents]
claude = "docker exec -i myproject-env claude --dangerously-skip-permissions"
codex = "docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox"
gemini = "docker exec -i myproject-env gemini --yolo"
EOF
```

4. **Use Agents with NTM**
```shell
ntm spawn myproject --cc=2 --cod=1 --gmi=1
```

## Dockerfile Templates

### Base Template (Minimal)
```dockerfile
FROM ruby:latest

# System dependencies
RUN apt-get update && apt-get install -y \
    curl git build-essential ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Claude Code
RUN curl -fsSL https://claude.ai/install.sh | sh

# Install Bun (for Codex/Gemini)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

# Install Codex and Gemini
RUN bun install -g @openai/codex @google/gemini-cli

# Declare volumes for persistent data
VOLUME ["/var/log/app", "/var/cache/app", "/tmp/app-data"]

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

### Python Project Template
```dockerfile
FROM ubuntu:24.04

# System dependencies
RUN apt-get update && apt-get install -y \
    curl git build-essential ca-certificates \
    python3 python3-pip python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Install agents
RUN curl -fsSL https://claude.ai/install.sh | sh
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"
RUN bun install -g @openai/codex @google/gemini-cli

# Python environment
RUN python3 -m pip install --upgrade pip setuptools wheel

# Add common Python tools
RUN pip install pytest black flake8 mypy

# Declare volumes for persistent data
VOLUME ["/var/log/app", "/root/.cache/pip", "/workspace/.venv"]

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

### Node.js Project Template
```dockerfile
FROM ubuntu:24.04

# System dependencies
RUN apt-get update && apt-get install -y \
    curl git build-essential ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt-get install -y nodejs

# Install agents
RUN curl -fsSL https://claude.ai/install.sh | sh
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"
RUN bun install -g @openai/codex @google/gemini-cli

# Add common Node.js tools
RUN npm install -g typescript eslint prettier

# Declare volumes for persistent data
VOLUME ["/var/log/app", "/root/.npm", "/workspace/node_modules"]

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

### Full Stack Template (Python + Node.js + Database)
```dockerfile
FROM ubuntu:24.04

# System dependencies
RUN apt-get update && apt-get install -y \
    curl git build-essential ca-certificates \
    python3 python3-pip python3-venv \
    postgresql-client redis-tools \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt-get install -y nodejs

# Install agents
RUN curl -fsSL https://claude.ai/install.sh | sh
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"
RUN bun install -g @openai/codex @google/gemini-cli

# Python tools
RUN pip install pytest black flake8 mypy django fastapi

# Node.js tools
RUN npm install -g typescript eslint prettier

# Declare volumes for persistent data (all application types)
VOLUME ["/var/log/app", "/root/.cache/pip", "/root/.npm", "/workspace/node_modules", "/workspace/.venv", "/var/lib/postgresql/data", "/var/lib/redis"]

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

## Container Lifecycle Management

### Starting Project Containers

**Option 1: Manual Container Management**
```shell
# Build image
docker build -t myproject-env .

# Start container with restart policy and persistence
docker run -d --name myproject-env \
  --restart=unless-stopped \
  -v "$(pwd):/workspace" \
  -v ~/.claude:/root/.claude \
  -v ~/.codex:/root/.codex \
  -v ~/.config/gemini:/root/.config/gemini \
  myproject-env

# Check status
docker ps | grep myproject-env
```

**Option 2: Docker Compose (Recommended)**
```yaml
# docker-compose.yml
version: '3.8'
services:
  agents:
    build: .
    container_name: myproject-env
    restart: unless-stopped
    volumes:
      - .:/workspace
      - ${HOME}/.claude:/root/.claude
      - ${HOME}/.codex:/root/.codex
      - ${HOME}/.config/gemini:/root/.config/gemini
    working_dir: /workspace
    command: tail -f /dev/null
```

```shell
# Start with compose
docker-compose up -d

# Stop
docker-compose down
```

### Container Management Commands

```shell
# Check if container is running
docker ps | grep myproject-env

# Start stopped container
docker start myproject-env

# Stop running container
docker stop myproject-env

# Remove container (will need to recreate)
docker rm myproject-env

# View container logs
docker logs myproject-env

# Access container shell for debugging
docker exec -it myproject-env bash

# Rebuild after Dockerfile changes
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

## Container Data Persistence

### Why Container Persistence Matters

**By default, Docker containers are ephemeral** - all data written inside the container is lost when the container is removed. For agent-flywheel workflows, this can impact:

- **Application data**: Databases, logs, generated files, caches
- **Development state**: Build artifacts, downloaded dependencies, IDE settings
- **Process data**: Long-running services, background tasks, temporary files

**Critical Scenarios Where Data Loss Occurs:**
- Container crashes and needs recreation (`docker rm` + `docker run`)
- System restarts without restart policy
- Manual cleanup or upgrades
- Docker daemon restarts

### Container Restart Policies

Always use `--restart=unless-stopped` to ensure containers survive system restarts:

```shell
# Manual container with restart policy
docker run -d --name myproject-env \
  --restart=unless-stopped \
  -v "$(pwd):/workspace" \
  myproject-env
```

**Restart Policy Options:**
- `no` - Never restart (default, not recommended)
- `on-failure` - Restart only if container exits with error
- `always` - Always restart (may interfere with manual stops)
- `unless-stopped` - ⭐ **Recommended**: Restart unless manually stopped

### Persistent Data Strategies

#### 1. Named Volumes (Recommended for Application Data)

For databases, caches, and application-generated content:

```yaml
# docker-compose.yml with named volumes
version: '3.8'
services:
  agents:
    build: .
    container_name: myproject-env
    restart: unless-stopped
    volumes:
      # Project code (bind mount)
      - .:/workspace
      # Agent credentials (bind mounts)
      - ${HOME}/.claude:/root/.claude
      - ${HOME}/.codex:/root/.codex
      - ${HOME}/.config/gemini:/root/.config/gemini
      # Persistent application data (named volumes)
      - postgres_data:/var/lib/postgresql/data
      - redis_data:/var/lib/redis
      - app_logs:/var/log/app
      - node_modules:/workspace/node_modules
    working_dir: /workspace

volumes:
  postgres_data:
  redis_data:
  app_logs:
  node_modules:
```

#### 2. Bind Mounts (For Development Files)

For files you want to access from the host:

```shell
# Database data directory on host
mkdir -p ./data/{postgres,redis,logs}

docker run -d --name myproject-env \
  --restart=unless-stopped \
  -v "$(pwd):/workspace" \
  -v "$(pwd)/data/postgres:/var/lib/postgresql/data" \
  -v "$(pwd)/data/redis:/var/lib/redis" \
  -v "$(pwd)/data/logs:/var/log/app" \
  myproject-env
```

### Database Persistence Examples

#### PostgreSQL Container with Persistence

```yaml
version: '3.8'
services:
  agents:
    build: .
    container_name: myproject-env
    restart: unless-stopped
    volumes:
      - .:/workspace
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myproject
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: devpass
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    container_name: myproject-postgres
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myproject
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: devpass

volumes:
  postgres_data:
```

#### Redis Cache with Persistence

```yaml
version: '3.8'
services:
  agents:
    build: .
    depends_on:
      - redis
    volumes:
      - .:/workspace
      - redis_data:/data

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

volumes:
  redis_data:
```

### Application Data Management

#### Where to Store Different Types of Data

```dockerfile
# In your Dockerfile, declare volumes for persistent directories
FROM ubuntu:24.04

# ... agent installation ...

# Declare persistent directories
VOLUME ["/var/log/app", "/var/cache/app", "/opt/app/data"]

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

#### Best Practices for Application Data

1. **Logs**: Mount `/var/log/app` to named volume or bind mount
2. **Databases**: Always use named volumes for database directories
3. **Cache**: Use named volumes for cache directories (Redis, npm cache, etc.)
4. **Build artifacts**: Consider named volumes for `node_modules`, `.venv`, etc.
5. **Temporary files**: Use `/tmp` (ephemeral) or mounted temp directory

### Backup and Recovery

#### Backing Up Named Volumes

```shell
# List all volumes
docker volume ls

# Backup a named volume
docker run --rm -v myproject_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz -C /data .

# Backup all application data
docker-compose down
docker run --rm \
  -v myproject_postgres_data:/data/postgres \
  -v myproject_redis_data:/data/redis \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/full_backup_$(date +%Y%m%d).tar.gz -C /data .
docker-compose up -d
```

#### Restoring from Backup

```shell
# Stop the application
docker-compose down

# Restore named volume from backup
docker run --rm -v myproject_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/postgres_backup_20241215.tar.gz -C /data

# Start the application
docker-compose up -d
```

#### Data Migration Between Environments

```shell
# Export data from source environment
docker run --rm -v source_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/migrate_data.tar.gz -C /data .

# Import data to target environment
docker run --rm -v target_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/migrate_data.tar.gz -C /data
```

### Common Persistence Anti-Patterns

❌ **Don't do this:**
```shell
# No restart policy - container won't survive reboot
docker run -d myproject-env

# Data written to container filesystem - lost when container removed
# (no volume mounts for application data)

# Manual database setup inside ephemeral container
docker exec -it myproject-env initdb
```

✅ **Do this instead:**
```shell
# Restart policy + proper volume mounts
docker run -d --restart=unless-stopped \
  -v postgres_data:/var/lib/postgresql/data \
  myproject-env

# External database with named volumes
docker-compose up -d  # with postgres service and volumes
```

### Data Loss Prevention Checklist

Before making changes to containers:

- [ ] Identify what data needs to persist (databases, logs, config, uploads)
- [ ] Configure appropriate volume mounts (named volumes or bind mounts)
- [ ] Set restart policy: `--restart=unless-stopped`
- [ ] Test persistence: stop/start container and verify data remains
- [ ] Document backup/restore procedures
- [ ] Consider: What happens during upgrades or migrations?

**⚠️  Warning Signs of Data Loss Risk:**
- No volume mounts for application data directories
- No restart policy configured
- Database/cache running inside ephemeral container filesystem
- Using `docker-compose down` without understanding persistence implications
- No backup procedures documented

## NTM Configuration

### Project-Specific Configuration

Create `.ntm/config.toml` in your project root:

```toml
[agents]
# Standard agent commands with docker exec
claude = "docker exec -i myproject-env claude --dangerously-skip-permissions"
codex = "docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox"
gemini = "docker exec -i myproject-env gemini --yolo"

# Optional: Custom agent configurations
[agents.aliases]
cc = "docker exec -i myproject-env claude --dangerously-skip-permissions"
cod = "docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox"
gmi = "docker exec -i myproject-env gemini --yolo"

# Optional: Model-specific overrides
[agents.models]
claude-opus = "docker exec -i myproject-env claude --dangerously-skip-permissions --model claude-3-opus-20240229"
codex-latest = "docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox -m gpt-4"
```

### Dynamic Container Names

For projects with multiple environments:

```toml
[agents]
claude = "docker exec -i ${CONTAINER_NAME:-myproject-env} claude --dangerously-skip-permissions"
codex = "docker exec -i ${CONTAINER_NAME:-myproject-env} codex --dangerously-bypass-approvals-and-sandbox"
gemini = "docker exec -i ${CONTAINER_NAME:-myproject-env} gemini --yolo"
```

Usage:
```shell
# Use default container
ntm spawn myproject --cc=2

# Use custom container
CONTAINER_NAME=myproject-dev ntm spawn myproject-dev --cc=2
```

## Credential Mounting

### Standard Credential Paths

Mount agent credentials from host to container:

```shell
docker run -d --name myproject-env \
  -v "$(pwd):/workspace" \
  -v ~/.claude:/root/.claude \
  -v ~/.codex:/root/.codex \
  -v ~/.config/gemini:/root/.config/gemini \
  myproject-env
```

### User ID Mapping (Linux)

Avoid permission issues by matching user IDs:

```dockerfile
FROM ubuntu:24.04

# Create user with host UID
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN groupadd -g ${GROUP_ID} developer && \
    useradd -m -u ${USER_ID} -g ${GROUP_ID} -s /bin/bash developer

# Install agents as user
USER developer
WORKDIR /home/developer
RUN curl -fsSL https://claude.ai/install.sh | sh
# ... rest of agent installation

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

Build with host UID:
```shell
docker build --build-arg USER_ID=$(id -u) --build-arg GROUP_ID=$(id -g) -t myproject-env .

docker run -d --name myproject-env \
  -v "$(pwd):/workspace" \
  -v ~/.claude:/home/developer/.claude \
  -v ~/.codex:/home/developer/.codex \
  -v ~/.config/gemini:/home/developer/.config/gemini \
  myproject-env
```

### macOS Volume Performance

For better performance on macOS:

```shell
docker run -d --name myproject-env \
  -v "$(pwd):/workspace:cached" \
  -v ~/.claude:/root/.claude:delegated \
  -v ~/.codex:/root/.codex:delegated \
  -v ~/.config/gemini:/root/.config/gemini:delegated \
  myproject-env
```

## Integration with bd init

When initializing a new beads project with Docker support:

1. **Initialize beads tracking**
```shell
bd init
```

2. **Create Docker environment** (optional step)
```shell
# Interactive setup script (future enhancement)
bd docker init

# Manual setup
touch Dockerfile docker-compose.yml
mkdir -p .ntm
```

3. **Add Docker files to git**
```shell
git add Dockerfile docker-compose.yml .ntm/config.toml
git commit -m "Add Docker environment for agents"
```

## Project Workflow Example

### Setting Up a New Python Project

1. **Create project structure**
```shell
mkdir ml-project && cd ml-project
bd init
```

2. **Create Dockerfile**
```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    curl git build-essential ca-certificates \
    python3 python3-pip python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Install agents
RUN curl -fsSL https://claude.ai/install.sh | sh
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"
RUN bun install -g @openai/codex @google/gemini-cli

# Python ML tools
RUN pip install pandas numpy scikit-learn jupyter

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

3. **Create docker-compose.yml**
```yaml
version: '3.8'
services:
  agents:
    build: .
    container_name: ml-project-env
    restart: unless-stopped
    volumes:
      - .:/workspace
      - ${HOME}/.claude:/root/.claude
      - ${HOME}/.codex:/root/.codex
      - ${HOME}/.config/gemini:/root/.config/gemini
      # ML-specific persistent data
      - ml_jupyter_data:/root/.jupyter
      - ml_cache:/root/.cache
      - ml_models:/workspace/models
      - ml_datasets:/workspace/data
    working_dir: /workspace

volumes:
  ml_jupyter_data:
  ml_cache:
  ml_models:
  ml_datasets:
```

4. **Configure NTM**
```shell
mkdir -p .ntm
cat > .ntm/config.toml << 'EOF'
[agents]
claude = "docker exec -i ml-project-env claude --dangerously-skip-permissions"
codex = "docker exec -i ml-project-env codex --dangerously-bypass-approvals-and-sandbox"
gemini = "docker exec -i ml-project-env gemini --yolo"
EOF
```

5. **Start environment and work**
```shell
docker-compose up -d
ntm spawn ml-project --cc=2 --cod=1
```

## Validation and Testing

After setting up your Docker environment, verify everything works correctly:

### Container Verification
```shell
# Check container is running
docker ps | grep myproject-env

# Verify workspace mount
docker exec myproject-env ls -la /workspace

# Verify credential mounts
docker exec myproject-env ls -la /root/.claude
docker exec myproject-env ls -la /root/.codex
docker exec myproject-env ls -la /root/.config/gemini
```

### Agent Installation Verification
```shell
# Check agents are installed
docker exec myproject-env which claude
docker exec myproject-env which codex
docker exec myproject-env which gemini

# Check agent versions
docker exec myproject-env claude --version
docker exec myproject-env codex --version
docker exec myproject-env gemini --version
```

### Agent Authentication Test
```shell
# Test Claude authentication
docker exec -i myproject-env claude --dangerously-skip-permissions "respond with OK"

# Test Codex authentication
docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox "respond with OK"

# Test Gemini authentication
docker exec -i myproject-env gemini --yolo "respond with OK"
```

### NTM Integration Test
```shell
# Spawn a test session with one agent
ntm spawn myproject --cc=1

# Verify session created
ntm list

# Clean up test session
ntm kill myproject
```

## Troubleshooting

### Container Not Running
```shell
# Check container status
docker ps -a | grep myproject-env

# Start if stopped
docker start myproject-env

# Recreate if failed
docker rm myproject-env
docker-compose up -d
```

### Permission Issues
```shell
# Check volume mounts
docker exec myproject-env ls -la /workspace
docker exec myproject-env ls -la /root/.claude

# Fix with user ID mapping (see User ID Mapping section above)
```

### Agent Not Found in Container
```shell
# Check agent installation
docker exec myproject-env which claude
docker exec myproject-env claude --version

# Reinstall if missing
docker exec myproject-env curl -fsSL https://claude.ai/install.sh | sh
```

### Credential Issues
```shell
# Verify credentials are mounted
docker exec myproject-env ls -la /root/.claude
docker exec myproject-env ls -la /root/.codex
docker exec myproject-env ls -la /root/.config/gemini

# Test agent authentication
docker exec myproject-env claude auth status
```

### Network Connectivity
```shell
# Test internet access
docker exec myproject-env curl -I https://api.anthropic.com

# Check DNS resolution
docker exec myproject-env nslookup api.anthropic.com
```

### NTM Configuration Issues
```shell
# Verify config syntax
ntm config validate

# Test agent commands manually
docker exec -i myproject-env claude --dangerously-skip-permissions
```

### Data Persistence Issues

#### Data Lost After Container Restart
```shell
# Check if volumes are properly mounted
docker inspect myproject-env | grep -A 10 '"Mounts"'

# Verify data directories exist and have data
ls -la ./data/  # for bind mounts
docker volume ls | grep myproject  # for named volumes

# If data is missing, check if restart policy is configured
docker inspect myproject-env | grep RestartPolicy

# Recreate container with proper volume mounts and restart policy
docker stop myproject-env
docker rm myproject-env
# Then rerun docker run command with --restart=unless-stopped and all volume mounts
```

#### Database Connection Failures After Container Recreation
```shell
# Check if database data volume is properly configured
docker volume inspect myproject_postgres_data

# Verify database files exist in volume
docker run --rm -v myproject_postgres_data:/data ubuntu ls -la /data

# If database volume is empty, restore from backup
docker run --rm -v myproject_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/postgres_backup_YYYYMMDD.tar.gz -C /data
```

#### Permission Denied on Data Directories
```shell
# Check ownership of data directories (bind mounts)
ls -la ./data/

# Fix permissions for bind-mounted data directories
sudo chown -R $(id -u):$(id -g) ./data/

# For PostgreSQL, use correct user ID
sudo chown -R 999:999 ./data/postgres

# For Redis, use correct user ID
sudo chown -R 999:999 ./data/redis
```

#### Application Data Not Persisting
```shell
# Identify where your application writes data
docker exec myproject-env find / -name "*.db" -o -name "*.log" 2>/dev/null

# Check if those directories are mounted as volumes
docker inspect myproject-env | grep -A 20 '"Mounts"'

# Add volume mounts for application data directories
# Update docker-compose.yml or docker run command
```

#### Named Volume Not Found
```shell
# List all volumes
docker volume ls

# Create missing volume
docker volume create myproject_postgres_data

# If volume was accidentally deleted, restore from backup
docker volume create myproject_postgres_data
docker run --rm -v myproject_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/postgres_backup_YYYYMMDD.tar.gz -C /data
```

#### Container Exits After System Restart
```shell
# Check restart policy
docker inspect myproject-env | grep RestartPolicy

# Update restart policy on existing container
docker update --restart=unless-stopped myproject-env

# Or recreate with proper restart policy
docker stop myproject-env
docker rm myproject-env
docker run -d --name myproject-env --restart=unless-stopped [other options...]
```

**Common Data Persistence Problems:**

| Problem | Cause | Solution |
|---------|-------|----------|
| Data disappears after `docker rm` | No volume mounts configured | Add `-v` flags for data directories |
| Container doesn't start after reboot | No restart policy | Add `--restart=unless-stopped` |
| Database connection errors | Database data not persisted | Mount `/var/lib/postgresql/data` |
| Permission denied on files | Wrong ownership on bind mounts | Use `chown` to fix permissions |
| Application logs missing | Logs written to ephemeral storage | Mount `/var/log/app` directory |
| Cache rebuilt every restart | Node modules not persisted | Mount `node_modules` as named volume |

## Best Practices

### Container Management
- **Use Docker Compose** for easier container lifecycle management
- **Name containers consistently** using project names
- **Keep containers running** with `tail -f /dev/null` or similar
- **Monitor container health** with `docker ps` and logs

### Data Persistence
- **Always use restart policies** (`--restart=unless-stopped`) to survive system reboots
- **Identify persistent data early** (databases, logs, cache, generated files)
- **Use named volumes for application data** (databases, caches) and bind mounts for development files
- **Document what needs persistence** in your project README
- **Test persistence** by stopping/starting containers and verifying data remains
- **Regular backups** of critical data volumes before major changes
- **Understand data loss scenarios** (container removal, volume deletion, system crashes)
- **Never assume data persists** without explicit volume configuration

### Security
- **Mount only necessary credentials** (`.claude`, `.codex`, `.config/gemini`)
- **Use read-only mounts** where possible for system files
- **Avoid exposing ports** unless required for the project
- **Regular image updates** for security patches

### Performance
- **Use volume caching** on macOS (`:cached`, `:delegated`)
- **Match user IDs** on Linux to avoid permission overhead
- **Minimize image layers** with multi-command RUN statements
- **Use .dockerignore** to exclude unnecessary files

### Team Collaboration
- **Commit Dockerfile and docker-compose.yml** to version control
- **Include .ntm/config.toml** in git for consistent agent setup
- **Document project-specific setup** in README.md
- **Use consistent container naming** across team members

### Maintenance
- **Rebuild images** after dependency changes
- **Clean up unused containers** periodically with `docker system prune`
- **Version your images** for reproducible environments
- **Test agent functionality** after container updates

This per-project Docker workflow provides consistent, isolated environments for agent-flywheel development while maintaining seamless integration with NTM and the broader toolchain.