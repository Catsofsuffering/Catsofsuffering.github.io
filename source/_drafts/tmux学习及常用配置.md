---
title: tmux学习及常用配置
tags:
---

## Tmux 配置

``` bash
#!/bin/bash

# change directory
if [ -n "$1" ]; then
    cd "$1"
fi
# make a new Tmux session
tmux new-session -d -s Boxwalk

# split the window
tmux split-window -v -p 50
tmux split-window -h

# select the upper pane
tmux select-pane -t 0

# attach the session
tmux attach-session -t Boxwalk
```
