# tmux

[[cheatsheet]] [[references]]

## Install

mac
```
brew install tmux
```
linux
```
apt-get install tmux
```
## tmux layers
```
Session Layer
|- Window Layer
   |- Pane Layer
```

## Command line

**Start**
```
tmux
```
**Create Session**
```
tmux new -s <sessionName>
```
**List All Sessions**
```
tmux ls
```
**Attach to the latest session**
```
tmux a
```
**Attach to the target session `<sessionName>`**
```
tmux a -t <sessionName>
```
**Kill the most recent session**
```
tmux kill-session
```
**Kill the session `<sessionName>`**
```
tmux kill-session -t <sessionName>
```
Kill all
```
tmux kill-server
```

## Basic Commands

| Command               | Description                 |
| --------------------- | --------------------------- |
| `Ctrl + b`            | tmux default leader |
| `<leader>` `d`          | Detach from current session |
| `<leader>` `c`          | Create new window           |
| `<leader>` `&`          | Close window                |
| `<leader>` `x`          | Close pane                  |
| `<leader>` `%`          | Split window horizontally   |
| `<leader>` `"`          | Split window vertically     |
| `<leader>` `n`          | Switch to next window       |
| `<leader>` `p`          | Switch to prev window       |
| `<leader>` `<num>`      | Switch window by index      |
| `<leader>` `<arrow>`    | Switch pane by arrow        |
| `<leader>` `q<num>`     | Switch pane by index        |
| `<leader>` `w`          | List sessions               |
| `<leader>` `s`          | List windows                |
| `<leader>` `:<command>` | Execute command             |

## Copy & Paste

| Command      | Description |
| ------------ | ----------- |
| `<leader>` `[` | Copy        |
| `<Space>`    | Start copy  |
| `<Enter>`    | Finish copy |
| `<leader>` `]` | Paste       |
