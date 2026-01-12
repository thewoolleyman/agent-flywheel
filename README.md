# agent-flywheel

Notes on setting up and using https://agent-flywheel.com/

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
