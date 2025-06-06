---
date: 2025-05-30
tags:
  - cheat-sheet
hubs:
  - "[[tmux]]"
  - "[[linux]]"
urls:
  - https://gist.github.com/MohamedAlaa/2961058
  - https://tmuxcheatsheet.com/
---

# tmux cheat sheet

## Outside tmux

New session

```bash
tmux
tmux new -s myname
```

Attach to existing session

```bash
tmux a
tmux a -t myname
tmux a -t myname -c /path/to/workdir
```

List, kill, detach sessions

```bash
tmux ls
tmux kill-session -t myname

tmux detach
```

## Inside tmux

Commands below must be proceeded by the bind key. The default is CTRL-b. My bind
key is "CTRL-b".

### General

```
?   list all shortcuts
:   run named tmux commands
```

### Panes

```
|   vertical split
-   horizontal split
o   swap panes
;   toggle last active pane
q   show pane numbers
x   kill pane
⍽   space - toggle between layouts
h   move the current pane left
l   move the current pane right

<C-c>↕/↔            resize pane
:break-pane -t :    break pane into new window
```

### Windows

```
c   create window
w   list windows
n   next window
p   previous window
f   find window
,   name window
.   move window or reorder
&   kill window
```

### Sessions

```
s   list sessions
$   name session
d   detach session
```

### Custom bindings (`~/.tmux.conf`)

```
h   move to left pane
j   move to lower pane
k   move to upper pane
l   move to right pane
-   re-number windows (change order)
```
