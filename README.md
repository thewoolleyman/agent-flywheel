# agent-flywheel

Notes on setting up and using https://agent-flywheel.com/

## Other docs and notes

- [Other notes](other-notes.md)
- [GitLab-specific setup](gitlab-setup.md)

## Installation, Doctor, and Update

### Update vs. Re-install

**Use `acfs-update`** for routine maintenance - keeping your existing installation current with the latest versions of agents, runtimes, and tools. Run this regularly (daily/weekly).

**Use re-install** when:
- Initial installation failed partway through and you need to resume/retry
- You want to re-run all phases (`--force-reinstall`)
- The ACFS installer itself has been updated with new features or phases
- You're switching modes (vibe ↔ safe)

Re-install is **non-destructive** - it preserves:
- User home directories and all contents
- Project files in `/data/projects`
- SSH keys and configs
- PostgreSQL databases
- Existing dotfiles and configurations

The `--force-reinstall` flag only resets the phase completion tracking (backs up `~/.acfs/state.json`), allowing all phases to re-run.

### Re-install

Run from the canonical repo (not fork):

```shell
curl --proto '=https' --proto-redir '=https' -fsSL 'https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh' | bash -s -- --mode vibe --yes --force-reinstall
```

Options:
- `--mode vibe` - Passwordless sudo, relaxed security (default for solo devs)
- `--mode safe` - Standard sudo, stricter security
- `--yes` - Non-interactive, skip prompts
- `--force-reinstall` - Fresh start, ignore previous installation state
- `--resume` - Resume from last checkpoint (auto-detected if previous install exists)

### Doctor

Check system health:

```shell
acfs doctor                   # Quick health check (existence checks only)
acfs doctor --deep            # Full functional tests (auth, connections)
acfs doctor --deep --no-cache # Force fresh deep checks (skip 5-min cache)
acfs doctor --json            # JSON output for tooling
```

Deep checks validate:
- Agent authentication (claude, codex, gemini)
- Database connectivity (PostgreSQL)
- Cloud CLI authentication (vault, wrangler, etc.)

### Update

Update all ACFS components:

```shell
acfs-update                   # Standard update (apt, runtimes, shell, agents, cloud)
acfs-update --stack           # Include Dicklesworthstone stack tools
acfs-update --dry-run         # Preview changes without making them
acfs-update --yes --quiet     # Automated/cron mode
```

Category options:
- `--apt-only` - Only system packages
- `--agents-only` - Only coding agents (Claude, Codex, Gemini)
- `--cloud-only` - Only cloud CLIs (Wrangler, Supabase, Vercel)
- `--shell-only` - Only shell tools (OMZ, P10K, plugins, Atuin, Zoxide)
- `--runtime-only` - Only runtimes (Bun, Rust, uv, Go)

Skip options: `--no-apt`, `--no-agents`, `--no-cloud`, `--no-shell`, `--no-runtime`

## General usage

### Coding Agents

Three agents are installed with power-mode aliases:

| Agent | Alias | Expands To |
|-------|-------|------------|
| Claude Code | `cc` | `NODE_OPTIONS="--max-old-space-size=32768" ENABLE_BACKGROUND_TASKS=1 claude --dangerously-skip-permissions` |
| Codex CLI | `cod` | `codex --dangerously-bypass-approvals-and-sandbox` |
| Gemini CLI | `gmi` | `gemini --yolo` |

**Login:**
```shell
claude auth login              # Anthropic - follow browser link
codex login --device-auth      # OpenAI - enable in ChatGPT Settings → Security first
gemini                         # Google - follow prompts
```

**Test:**
```shell
cc "Hello! Confirm you're working."
cod "Hello! Confirm you're working."
gmi "Hello! Confirm you're working."
```

### CAAM (Coding Agent Account Manager)

Backup and switch agent credentials. Useful for managing rate limits:

```shell
caam backup claude my-main-account    # Backup current credentials
caam backup codex my-main-account
caam backup gemini my-main-account
caam ls                               # List all backups
caam status                           # See current accounts
caam activate claude other-account    # Switch to different account
```

### NTM (Named Tmux Manager)

Orchestrate multiple agents in organized tmux sessions:

```shell
ntm tutorial                          # Interactive tutorial
ntm deps -v                           # Check dependencies
ntm spawn myproject --cc=2 --cod=1 --gmi=1  # Create session with agents
ntm list                              # List sessions
ntm attach myproject                  # Attach to session
ntm send myproject "Analyze this codebase"  # Send to ALL agents
ntm send myproject --cc "Focus on API"      # Send to specific agent type
ntm palette                           # Browse pre-built prompts
ntm palette myproject --send          # Select prompt and send to session
```

**Inside NTM session:**

| Keys | Action |
|--------------------------|---------------------|
| `Ctrl+a` then `n` | Next window |
| `Ctrl+a` then `p` | Previous window |
| `Ctrl+a` then `h/j/k/l` | Move between panes |
| `Ctrl+a` then `z` | Zoom current pane |

### UBS (Ultimate Bug Scanner)

Run comprehensive static analysis to catch issues before committing:

```shell
ubs .                                 # Scan current directory
```

### CASS (Coding Agent Session Search)

Search across all agent session history:

```shell
cass                                  # Opens TUI for searching
```

### CM (CASS Memory)

Build persistent procedural memory that gives agents context from past work:

```shell
cm context "Building an API"          # Get relevant memories for context
cm context "auth system" --json       # JSON output for piping to agents
cm reflect                            # Update procedural memory from sessions
```

### bv/bd (Beads Viewer)

Track tasks and issues with a kanban-style interface:

```shell
bv                                    # Opens TUI (kanban view)
bd init                               # Initialize beads in current project
bd ready                              # See tasks ready to work on
bd close <task-id>                    # Close a task
```

**Recommended setup for new projects:**
```shell
git branch beads-sync main            # Create dedicated sync branch
git push -u origin beads-sync
bd config set sync.branch beads-sync  # Avoid worktree conflicts
```

### am (Agent Mail)

Multi-agent coordination server that lets agents share context with each other:

```shell
am                                    # Start the agent mail server
```

### SLB (Simultaneous Launch Button)

Optional safety guardrails with two-person rule for dangerous commands:

```shell
slb                                   # Review dangerous operations
```

### Typical Workflow

```shell
# 1. Plan work
bv                                    # Check tasks
bd ready                              # See what's ready

# 2. Start agents
ntm spawn myproject --cc=2 --cod=1

# 3. Set context from memory
cm context "Implementing auth" --json

# 4. Send initial prompt
ntm send myproject "Implement user auth. Context: [paste cm output]"

# 5. Monitor
ntm attach myproject

# 6. Scan before commit
ubs .

# 7. Update memory
cm reflect

# 8. Close task
bd close <task-id>
```

## Running Agents in Custom Containers

Run agents inside Docker containers with project-specific environments (databases, language runtimes, system dependencies). This gives agents access to a fully-configured development environment.

### Method 1: Docker Exec (Quick Tasks)

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

### Method 2: SSH-Enabled Container with NTM

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

### Method 3: Custom NTM Agent Commands

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

### Method 4: Per-Project Container Config

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

### Mounting Credentials

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

### Container Best Practices

1. **Keep containers running** - Use `tail -f /dev/null` or a process manager
2. **Mount project directories** - Agents need access to your code
3. **Share credentials** - Mount `.claude`, `.codex`, `.config/gemini` from host
4. **Match user IDs** - Avoid permission issues by matching container user UID to host
5. **Use named containers** - Easier to reference in commands and configs

### Example: Python ML Project

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

## VPS provider info

- https://contabo.com/en-us/vps/
- https://new.contabo.com/servers/vps

## Fork of project

- https://github.com/thewoolleyman/agentic_coding_flywheel_setup
- PR: https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/pull/30
- Run from fork with:

```shell
export ACFS_REPO_OWNER=thewoolleyman && curl -fsSL "https://raw.githubusercontent.com/thewoolleyman/agentic_coding_flywheel_setup/main/install.sh" | bash -s -- --yes --force-reinstall
```

## Logging into VPN

- Add an /etc/hosts entry `agent-flywheel-host`
- For initial install, log in as `ssh root@agent-flywheel-host`
- After setup is complete, log in as `ssh ubuntu@agent-flywheel-host`
