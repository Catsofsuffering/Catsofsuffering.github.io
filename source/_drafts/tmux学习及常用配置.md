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

# enable mouse on
tmux set mouse on

# attach the session
tmux attach-session -t Boxwalk
```

## Go 环境变量配置

Kali终端为zshrc，只是在profile中添加环境变量无法生效，因此还需要更改zshrc的配置文件。

```
echo "export PATH=/usr/local/go/bin:$PATH" >> .zshrc
echo "export PATH=/home/kali/go/bin:$PATH" >> .zshrc
echo "export PATH=/usr/local/go/bin:$PATH" >> .profile
echo "export PATH=/home/kali/go/bin:$PATH" >> .profile
```