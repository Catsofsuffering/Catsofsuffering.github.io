---
title: Linux踩坑记录汇总
date: 2023-07-10 00:47:08
tags:
  - 备忘录
  - Linux
categories: 备忘录
keywords: Linux
description: 作者是 Linux 初学者，在学习使用 Linux 中难免会遇到一些比较棘手的问题，因此汇总起来方便查阅以及防止忘记，其中包含了 ss5 代理无法正常连接等问题。这篇文章会长期更新。
top_img: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
cover: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
---


## ss5 代理无法正常连接

### 问题详情

某个项目需要通过代理进入内网，然后对目标项目进行测试，但是 `socks5`  代理一直连接不上，初步怀疑是 ss5 服务挂掉了。

### 解决办法

幸好组长给了服务器的账号密码，所以直接上服务器排查：

```bash
# check ss5 conf
cat /etc/opt/ss5/ss5.conf
cat /etc/opt/ss5/ss5.conf | grep auth
# check ss5 passwd
cat /etc/opt/ss5/ss5.passwd
# check ss5 port
cat /etc/sysconfig/ss5
# view ss5 log
tail -f /var/log/ss5/ss5.log
# view service log
journalctl -u ss5
```

依次检查下面可能会出现问题的地方

1. `ss5` 服务配置是否错误；
2. 代理账号密码正确与否；
3. 服务是否正常启动；
4. 防火墙是否放行数据；
5. 云服务器防火墙是否正常放行数据；

最终定位错误原因，机器重启无法创建文件：

```
Can’t create pid file /var/run/ss5/ss5.pid
Can’t unlink pid file /var/run/ss5/ss5.pid
```

使用解决办法：手动创建文件夹，并将其写入到系统自启动脚本中，再将 `ss5` 服务设置为在系统启动时自动运行。

```bash
echo 'mkdir /var/run/ss5/' >> /etc/rc.d/rc.local ;\
chmod +x /etc/rc.d/rc.local ;\ 
/sbin/chkconfig ss5 on
```

## 知识点总结

### tail命令

tail命令是一个用于查看文件末尾内容的命令。它可以显示文件的最后几行，默认情况下显示文件的最后10行。常见的用法如下：

1. `tail filename`：显示文件的最后10行。
2. `tail -n N filename`：显示文件的最后N行，其中N为一个整数。
3. `tail -f filename`：实时追踪文件的变化，当文件新增内容时会自动显示新增的内容。
4. `tail -c N filename`：显示文件的最后N个字节，其中N为一个整数。
5. `tail -q filename1 filename2`：不显示文件名，多个文件之间不用空行分隔。
6. `tail -v filename1 filename2`：显示文件名，多个文件之间用空行分隔。

### journalctl命令

`journalctl` 命令是用于查看和管理 `Systemd` 日志的工具。`Systemd` 是 Linux 系统中常用的初始化系统和服务管理器。常见的用法如下：

1. 查看系统日志：`journalctl` 命令默认会显示当前系统日志的内容，通过添加参数可以限制显示的日志数量、时间范围等。
2. 过滤日志：`journalctl`命令能够可以根据时间范围、日志级别、特定单元（如服务或进程）等过滤显示的日志内容。
3. 跟踪日志：`journalctl -f` 可以实时跟踪并显示最新的日志消息。
4. 导出日志：`journalctl` 能将日志导出为文件。


### ss5 使用

