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

某个项目需要通过代理进入内网，然后对目标项目进行测试，但是 `socks5`  代理一直连接不上，初步怀疑是 ss5 服务<sup>[①](#ss5-服务)</sup>挂掉了。

### 解决办法

幸好组长给了服务器的账号密码，所以直接上服务器排查：<sup>[②](#tail命令)</sup><sup>[③](#journalctl命令)</sup>

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

## find 命令

```
find /etc/ -name "*" | xargs grep -ri "192.168.0.103" -l;
```
## `/dev/mapper/` 是什么

`/dev/mapper` 是 Linux 系统中用于 LVM（Logical Volume Manager）和 DM（Device Mapper）设备映射的设备路径。这个目录通常包含 LVM 创建的逻辑卷和 DM 创建的设备映射。

1. **LVM（Logical Volume Manager）：** LVM 是一种用于管理磁盘存储的软件，它允许将物理磁盘的空间抽象为逻辑卷（Logical Volumes），并允许在运行时调整这些卷的大小。在 LVM 中，逻辑卷的路径通常是 `/dev/mapper/vg_name-lv_name`，其中 `vg_name` 是卷组（Volume Group）的名称，`lv_name` 是逻辑卷的名称。

   例如：
   ```plaintext
   /dev/mapper/myvg-mylv
   ```

2. **DM（Device Mapper）：** Device Mapper 是 Linux 内核中的一个框架，用于在块设备和块设备之间创建映射。它允许在逻辑块设备和底层物理设备之间插入不同的层，从而支持各种高级存储技术。在 `/dev/mapper` 中，你可能会看到一些由 DM 创建的设备映射，这些设备映射的名称取决于其配置。

   例如：
   ```plaintext
   /dev/mapper/encrypted-home
   ```

总之，`/dev/mapper` 是一个包含 LVM 逻辑卷和 DM 设备映射的目录。在使用 LVM 或者其他设备映射技术时，这个目录下的设备路径会动态地生成。


## 注释

### ss5 服务

`ss5` 是一个开源的 SOCKS (SOCKet Secure) 代理服务器软件，提供 SOCKS v4 和 SOCKS v5 协议支持。它允许网络中的计算机通过代理服务器进行连接，并通过这个代理服务器进行通信，以实现一些网络访问的控制和安全性需求。

以下是 `ss5` 服务的一些主要特点和用途：

1. **SOCKS 代理服务：** `ss5` 提供 SOCKS 代理服务，可以为支持 SOCKS 协议的应用程序提供代理服务。这对于需要通过中间代理进行网络连接的应用程序很有用。

2. **支持 SOCKS v4 和 SOCKS v5：** `ss5` 同时支持 SOCKS v4 和 SOCKS v5 协议，这两者在功能和安全性上有一些区别。

3. **用户认证支持：** 可以配置 `ss5` 要求用户进行身份验证，以确保只有授权用户可以通过代理进行连接。

4. **访问控制：** `ss5` 允许管理员配置访问控制规则，限制哪些用户或哪些 IP 地址可以通过代理进行连接，以提高网络安全性。

5. **运行日志：** `ss5` 提供日志功能，记录代理服务器的运行状况，以便管理员监控和审计代理的使用情况。

6. **轻量级：** `ss5` 是一个相对轻量级的代理服务器，易于安装和配置。

以下是一个简单的示例，展示了如何通过 `ss5` 在本地启动 SOCKS 代理服务：

```bash
# 安装 ss5（使用适合您的包管理器）
sudo apt-get install ss5   # 对于 Debian/Ubuntu 系统

# 启动 ss5 服务
sudo /etc/init.d/ss5 start
```

请注意，这只是一个简单的示例。在生产环境中，您需要根据具体的需求和安全标准进行更详细的配置和管理。使用 SOCKS 代理服务器时，请确保您了解其安全性和合规性要求。