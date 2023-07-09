---
title: Vulnhub-KioptrixLevel1通关记录
tags:
---

## 靶机描述

靶机 IP :  10.8.12.128

>This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.

## 信息收集

### nmap扫描

按照常规的方法：通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息

下面是通过 TCP 扫描发现所收集到的信息：

```
┌──(kali㉿K4)-[~/Desktop/BoxWalk/Vulnhub/kioptrix/level1]                                └─$ sudo nmap -sT -p- --min-rate 10000 10.8.12.128 -oN nmapscan/portscan.txt
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


┌──(kali㉿K4)-[~/Desktop/BoxWalk/Vulnhub/kioptrix/level1]
└─$ sudo nmap -sT -p22,80,111,139,443,1024 -sV -sC -O --min-rate 10000 10.8.12.128 -oN nmapscan/servscan.txt
.......
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open  status      1 (RPC #100024)
.......

Linux 2.4.9 - 2.4.18 (likely embedded) (97%)......
```


### Web 目录扫描


```
/.html                (Status: 403) [Size: 269]
/index.html           (Status: 200) [Size: 2890]
/test.php             (Status: 200) [Size: 27]
/manual               (Status: 301) [Size: 294] [--> http://127.0.0.1/manual/]
/usage                (Status: 301) [Size: 293] [--> http://127.0.0.1/usage/]
/mrtg                 (Status: 301) [Size: 292] [--> http://127.0.0.1/mrtg/]
```

## 总结

### 知识点

#### RPC协议

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