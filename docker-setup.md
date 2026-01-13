# Running Agents in Custom Containers

Run agents inside Docker containers with project-specific environments (databases, language runtimes, system dependencies). This gives agents access to a fully-configured development environment.

## Method 1: Docker Exec (Quick Tasks)

For one-off tasks in an existing container:

1. Start your project container:
```shell
docker run -d --name myproject-env \
  -v /data/projects/myproject:/workspace \
  -w /workspace \
  myproject-image:latest \
  tail -f /dev/null
```

2. Run an agent inside the container:
```shell
docker exec -it myproject-env claude --dangerously-skip-permissions
```

3. Or create a wrapper alias:
```shell
alias cc-docker='docker exec -it myproject-env claude --dangerously-skip-permissions'
```

## Method 2: SSH-Enabled Container with NTM

For full NTM integration, create a container with SSH access and use NTM's `--ssh` flag.

1. Create a `Dockerfile` with SSH and agent dependencies:
```dockerfile
FROM ubuntu:24.04

# Install SSH and base tools
RUN apt-get update && apt-get install -y \
    openssh-server sudo curl git \
    && rm -rf /var/lib/apt/lists/*

# Create agent user with sudo
RUN useradd -m -s /bin/bash agent && \
    echo "agent ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set up SSH
RUN mkdir /var/run/sshd && \
    mkdir -p /home/agent/.ssh && \
    chown agent:agent /home/agent/.ssh

# Install your project dependencies here
# RUN apt-get install -y nodejs npm python3 ...
# RUN pip install ...

# Install agents (as agent user)
USER agent
WORKDIR /home/agent

# Install Claude Code
RUN curl -fsSL https://claude.ai/install.sh | sh

# Install Bun (for Codex/Gemini)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/home/agent/.bun/bin:$PATH"

# Install Codex and Gemini
RUN ~/.bun/bin/bun install -g @openai/codex @google/gemini-cli

USER root
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

2. Build and run the container:
```shell
docker build -t myproject-agent-env .

docker run -d --name myproject-agents \
  -p 2222:22 \
  -v /data/projects/myproject:/home/agent/project \
  -v /home/ubuntu/.claude:/home/agent/.claude \
  -v /home/ubuntu/.codex:/home/agent/.codex \
  -v /home/ubuntu/.config/gemini:/home/agent/.config/gemini \
  myproject-agent-env
```

3. Add your SSH key to the container:
```shell
docker exec myproject-agents bash -c \
  "cat >> /home/agent/.ssh/authorized_keys" < ~/.ssh/id_rsa.pub
```

4. Use NTM with the `--ssh` flag:
```shell
ntm spawn myproject --cc=2 --cod=1 --ssh=agent@localhost:2222
```

## Method 3: Custom NTM Agent Commands

Override agent commands in NTM config to run inside containers.

1. Edit NTM config:
```shell
ntm config edit
```

2. Modify the `[agents]` section to use docker exec:
```toml
[agents]
claude = "docker exec -i myproject-env claude --dangerously-skip-permissions{{if .Model}} --model {{shellQuote .Model}}{{end}}"
codex = "docker exec -i myproject-env codex --dangerously-bypass-approvals-and-sandbox{{if .Model}} -m {{shellQuote .Model}}{{end}}"
gemini = "docker exec -i myproject-env gemini --yolo{{if .Model}} --model {{shellQuote .Model}}{{end}}"
```

3. Spawn agents normally (commands now run in container):
```shell
ntm spawn myproject --cc=2 --cod=1
```

## Method 4: Per-Project Container Config

Use NTM project config to override agent commands per-project.

1. In your project directory:
```shell
cd /data/projects/myproject
ntm config project init
```

2. Edit `.ntm/config.toml`:
```toml
[agents]
claude = "docker exec -i myproject-env claude --dangerously-skip-permissions"
```

3. Agents spawned in this directory use the container.

## Mounting Credentials

When running agents in containers, mount credential directories from the host:

```shell
docker run -d --name myproject-env \
  -v /data/projects/myproject:/workspace \
  -v /home/ubuntu/.claude:/root/.claude \
  -v /home/ubuntu/.codex:/root/.codex \
  -v /home/ubuntu/.config/gemini:/root/.config/gemini \
  -w /workspace \
  myproject-image:latest
```

## Container Best Practices

1. **Keep containers running** - Use `tail -f /dev/null` or a process manager
2. **Mount project directories** - Agents need access to your code
3. **Share credentials** - Mount `.claude`, `.codex`, `.config/gemini` from host
4. **Match user IDs** - Avoid permission issues by matching container user UID to host
5. **Use named containers** - Easier to reference in commands and configs

## Example: Python ML Project

```dockerfile
FROM python:3.11

RUN pip install torch transformers pandas numpy jupyter

# Install Claude
RUN curl -fsSL https://claude.ai/install.sh | sh

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

```shell
# Build and run
docker build -t ml-env .
docker run -d --name ml-env \
  -v /data/projects/ml-project:/workspace \
  -v /home/ubuntu/.claude:/root/.claude \
  --gpus all \
  ml-env

# Use with agent
docker exec -it ml-env claude --dangerously-skip-permissions
```