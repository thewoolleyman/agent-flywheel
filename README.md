# agent-flywheel

Notes on setting up and using https://agent-flywheel.com/

## Other docs and notes

- [Other notes](other-notes.md)
- [GitLab-specific setup](gitlab-setup.md)
- [Docker setup for agents](docker-setup.md)

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

## Initial project setup

### For new projects

First, clone or create your project repository:

```shell
gh repo clone <owner/repo>            # Clone GitHub project
# OR
glab repo clone <owner/repo>          # Clone GitLab project
# OR
mkdir my-new-project && cd my-new-project && git init  # Create new project
```

Initialize beads issue tracking and configure git branches:

```shell
bd init                               # Initialize beads in current project
git branch beads-sync main            # Create dedicated sync branch
git push -u origin beads-sync
bd config set sync.branch beads-sync  # Avoid worktree conflicts
```

### For existing projects

If you're working on a project that already has agent-flywheel setup:

```shell
gh repo clone <owner/repo>            # Clone GitHub project
# OR
glab repo clone <owner/repo>          # Clone GitLab project
cd <project-directory>
git checkout beads-sync               # Switch to beads branch if it exists
bd ready                              # Check for ready tasks
```

### Verify setup

Test that your environment is properly configured:

```shell
acfs doctor                           # Quick health check
bd ready                              # Check beads status
bv                                    # Open kanban view
```

## Typical Workflow

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
