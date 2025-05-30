---
date: 2025-05-30
tags:
  -
hubs:
  - "[[]]"
urls:
  -
---

# Tmux: Terminal Multiplexer

# Tmux: Terminal Multiplexer

## Session, Windows and Panes inside tmux

We start by creating a tmux session, that session could have multiple windows.
Each window could be split into panes either horizontal or vertical.

Command Mode : `Ctrl+Shift+B` opens the command mode in the status bar where we
can right commands or use shortcuts instead if we don't want to type in the
whole command.

## Session Management:

- Session: A session is a workspace that can hold multiple Windows.

* Creating a new session

  - Type `new-session` or from command line type `tmux new -s [session-name]`.

* Attaching to a existing session

  - Type `tmux attach -t [session-name]` from the command line.

* Leave a session

  - Type `tmux detach`.

* Veiw all sessions from command line
  - Type `tmux ls` from command line or press `Ctrl+b+s`.

## Window Management:

- Window: A window in tmux is like a tab in our terminal session. Each window
  can run its own shell and have multiple panes inside it.

* Creating a new Window.

  - Type `new window` when in command mode to launch a new window
  - Shortcut to open a new window is `Ctrl+b+c`
  - Renaming a window by `Ctrl+b+,`

* Switching between windows

  - Switch between this windows using `Ctrl+b+[0-9]` to switch to the window
    numbered 0 throuh 9.
  - To see all your windows, `Ctrl+b+w`.
  - To toggle betwen windows using `Ctrl+b+n` for next window and `Ctrl+b+p` for
    previous windows.

* Closing window
  - Type `exit` in the shell to close the window

## Pane Management:

- Pane : A pane is a split region inside a tmux window where a shell runs.
  Splits can be done in horizontally or vertically.

* Creatin a new split
  - Type `split-window -h` in command mode to split window horizontally or press
    `Ctrl+b+-`.
  - Type `split-window -v` in command mode to split window vertically or press
    `Ctrl+b+|`.

## Copy pasting with tmux

- with the copy paste plugin installed, copy pasting is easy with a selection
  with the mouse.
- select with the use and use `Ctrl+u` or `Ctrl+d` to move up and down.

```bash
# for yanking

set -g @plugin 'tmux-plugins/tmux-yank'
```
