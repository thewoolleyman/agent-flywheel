# Chad's notes

General notes on things from https://agent-flywheel.com/learn that I didn't already know or
don't remember to use regularly.

## CLI commands

- `rg`: ripgrep
- `fd`: find files
- `w`: like `who` but with info on what users are doing

## tmux usage

NOTE: In ACFS, the prefix key is `Ctrl+a` (not the default `Ctrl+b`). All tmux commands start with the prefix.

### Concepts: Windows vs Panes

- **Window** = A full-screen tab. Only one window is visible at a time. Switch between them like browser tabs.
- **Pane** = A split within a window. Multiple panes are visible simultaneously.

```
┌─────────────────────────────┐
│  Window 1    Window 2   ... │  ← Windows (tabs)
├──────────────┬──────────────┤
│              │              │
│   Pane 1     │   Pane 2     │  ← Panes (splits)
│              │              │
├──────────────┴──────────────┤
│           Pane 3            │
└─────────────────────────────┘
```

### Command mode
- Enter command mode: `Ctrl+a` then `:`
- From there you can type any tmux command (e.g., `join-pane -s :2`)

### Session commands
- New named session: `tmux new -s myproject` (`ntm spawn myproject`)
- List sessions: `tmux ls` (`ntm list`)
- Attach to session: `tmux attach -t myproject` or just `tmux a` (`ntm attach myproject`)
- Detach: `Ctrl+a` then `d` (keeps session running)
- Kill session: `tmux kill-session -t myproject` (`ntm kill myproject`)

### Window commands
- New window: `Ctrl+a` then `c`
- Next window: `Ctrl+a` then `n`
- Previous window: `Ctrl+a` then `p`
- Go to window: `Ctrl+a` then `0-9`
- List windows interactively: `Ctrl+a` then `w`
- List windows (shell): `tmux list-windows`
- Window numbers also shown in the status bar at bottom

### Pane commands
- Split vertically: `Ctrl+a` then `|`
- Split horizontally: `Ctrl+a` then `-`
- Navigate panes: `Ctrl+a` then `h/j/k/l`
- Close pane: `Ctrl+a` then `x`

### Moving panes and windows
- Break pane to new window: `Ctrl+a` then `!`
- Join window as pane (in command mode): `join-pane -s :<window-number>`
  - Use colon prefix (`:2` not `2`) to specify current session
- Send current pane to another window: `join-pane -t :<target-window>`
- Or run directly from shell: `tmux join-pane -s :<window-number>`

### Resizing panes
Keyboard shortcut (if configured): `Ctrl+a` then `Alt+↑/↓/←/→`

In command mode (`Ctrl+a` then `:`):
- `resize-pane -D 10` — resize down
- `resize-pane -U 10` — resize up
- `resize-pane -L 10` — resize left
- `resize-pane -R 10` — resize right

Or from shell: `tmux resize-pane -D 10`

### Copy mode
- Enter copy mode: `Ctrl+a` then `[`
- Scroll: `j/k` or arrow keys
- Start selection: `v`
- Copy selection: `y`
- Exit: `q`
