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
FROM ubuntu:24.04

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
FROM ubuntu:24.04

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

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

## Container Lifecycle Management

### Starting Project Containers

**Option 1: Manual Container Management**
```shell
# Build image
docker build -t myproject-env .

# Start container
docker run -d --name myproject-env \
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
    volumes:
      - .:/workspace
      - ~/.claude:/root/.claude
      - ~/.codex:/root/.codex
      - ~/.config/gemini:/root/.config/gemini
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
    volumes:
      - .:/workspace
      - ~/.claude:/root/.claude
      - ~/.codex:/root/.codex
      - ~/.config/gemini:/root/.config/gemini
    working_dir: /workspace
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

## Best Practices

### Container Management
- **Use Docker Compose** for easier container lifecycle management
- **Name containers consistently** using project names
- **Keep containers running** with `tail -f /dev/null` or similar
- **Monitor container health** with `docker ps` and logs

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