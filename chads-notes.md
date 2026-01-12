# Chad's notes

General notes on things from https://agent-flywheel.com/learn that I didn't already know or
don't remember to use regularly.

## CLI commands

- `rg`: ripgrep
- `fd`: find files
- `w`: like `who` but with info on what users are doing

## tmux usage

NOTE: In ACFS, the prefix key is `Ctrl+a` (not the default `Ctrl+b`). All tmux commands start with the prefix.

### Session commands
- New named session: `tmux new -s myproject` (`ntm spawn myproject`)
- List sessions: `tmux ls` (`ntm list`)
- Attach to session: `tmux attach -t myproject` or just `tmux a` (`ntm attach myproject`)
- Detach: `Ctrl+a` then `d` (keeps session running)
- Kill session: `tmux kill-session -t myproject` (`ntm kill myproject`)

### Pane commands
- Split vertically: `Ctrl+a` then `|`
- Split horizontally: `Ctrl+a` then `-`
- Navigate panes: `Ctrl+a` then `h/j/k/l`
- Close pane: `Ctrl+a` then `x`

### Window commands
- New window: `Ctrl+a` then `c`
- Next window: `Ctrl+a` then `n`
- Previous window: `Ctrl+a` then `p`
- Go to window: `Ctrl+a` then `0-9`

### Copy mode
- Enter copy mode: `Ctrl+a` then `[`
- Scroll: `j/k` or arrow keys
- Start selection: `v`
- Copy selection: `y`
- Exit: `q`
