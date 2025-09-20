---
title: HTB-Administrator通关
tags:
---
## 条件

- As is common in real life Windows pentests, you will start the Administrator box with credentials for the following account: **Olivia / ichliebedich**
- Windows System

## 扫描

使用 `rustscan` 扫描：

```
rustscan --ulimit 5000 --timeout 10000 -a admini-htb -g | nmap -Pn -sV -sC -oN port.log -iL -


# Result

```

根据提示条件