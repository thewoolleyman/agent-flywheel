# agent-flywheel

Notes on setting up and using https://agent-flywheel.com/

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
curl --proto '=https' --proto-redir '=https' -fsSL 'https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh' | bash -s -- --mode vibe --yes
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
