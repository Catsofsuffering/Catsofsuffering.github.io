---
title: Vulnhub-SICKOSv1.1通关记录
tags:
  - vulnhub
  - SickOS
categories: WriteUp
keywords: vulnhub
description: 本文记录了作者在通关vulnhub靶机SICKOSv1.1中遇到的问题以及踩坑后的解决方案。
top_img: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
cover: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
date: 2023-09-11 15:32:20
---

## 靶机信息

靶机 IP ：192.168.213.130

靶机介绍：This CTF gives a clear analogy how hacking strategies can be performed on a network to compromise it in a safe environment. This vm is very similar to labs I faced in OSCP. The objective being to compromise the network/machine and gain Administrative/root privileges on them.

## 信息收集

### 端口发现和服务探测

使用 Nmap 基于 TCP 协议扫描端口：

```
$ sudo nmap -sT -p- --min-rate 10000 192.168.213.130 -oN nmapscan/portscan.txt
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-11 04:01 EDT
Nmap scan report for 192.168.213.130
Host is up (0.00061s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE  SERVICE
22/tcp   open   ssh
3128/tcp open   squid-http
8080/tcp closed http-proxy
MAC Address: 00:0C:29:74:56:35 (VMware)
```

发现 22，3128，8080 端口是打开的，进一步扫描对应端口的服务：

```
$ sudo nmap -sT -p 22,3128,8080 -sV -sC -O 192.168.213.130 -oN nmapscan/servscan.txt
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-13 01:52 EDT
Nmap scan report for 192.168.213.130
Host is up (0.00037s latency).

PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 093d29a0da4814c165141e6a6c370409 (DSA)
|   2048 8463e9a88e993348dbf6d581abf208ec (RSA)
|_  256 51f6eb09f6b3e691ae36370cc8ee3427 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.1.19
8080/tcp closed http-proxy
MAC Address: 00:0C:29:74:56:35 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.16 seconds
```

探测完服务和端口后，可以得出以下结论：
1. 3128端口运行着 Squid 服务<sup>[①](#Squid-服务)</sup>，其运行版本为 3.1.19。
2. 8080端口存在服务，但端口关闭，不允许外界访问

### 通过代理访问目标网页

尝试使用 `curl` 命令，通过代理的方式访问8080端口，但仍旧失败。

``` bash
curl --proxy http://192.168.213.130:3128 http://192.168.213.130:8080
```

根据[提醒](https://www.bilibili.com/video/BV1Em4y1A7XX?t=1058.7)，爆破网页目录试试

```
$ sudo dirb http://192.168.213.130 -p http://192.168.213.130:3128

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Oct 20 04:56:57 2023
URL_BASE: http://192.168.213.130/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
PROXY: http://192.168.213.130:3128

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.213.130/ ----
+ http://192.168.213.130/cgi-bin/ (CODE:403|SIZE:291)
+ http://192.168.213.130/connect (CODE:200|SIZE:109)
+ http://192.168.213.130/index (CODE:200|SIZE:21)
+ http://192.168.213.130/index.php (CODE:200|SIZE:21)
+ http://192.168.213.130/robots (CODE:200|SIZE:45)
+ http://192.168.213.130/robots.txt (CODE:200|SIZE:45)
+ http://192.168.213.130/server-status (CODE:403|SIZE:296)
-----------------
END_TIME: Fri Oct 20 04:57:00 2023
DOWNLOADED: 4612 - FOUND: 7
```

再次使用 `curl` 命令测试，发现能够正常访问目标网页了，成功通过代理绕过端口限制。

```bash
$ curl --proxy http://192.168.213.130:3128 http://192.168.213.130/index.php
<h1>
BLEHHH!!!
</h1>
```

配置浏览器 `SwitchOmega` 插件，根据目录爆破的结果，访问 `/robots` 文件，获得以下信息：

```
$ curl --proxy http://192.168.213.130:3128 http://192.168.213.130/robots   
User-agent: *
Disallow: /
Dissalow: /wolfcms
```

## 获取立足点

### 弱口令登录网站后台

根据上文获得的信息，直接访问 `/wolfcms` ，可以观察到如下页面。

![pic1](https://pic.imgdb.cn/item/65324464c458853aef8ab069.jpg)

使用 `searchsploit` 发现 Wolfcms存在历史漏洞，但是三个历史漏洞都缺乏利用条件，无法正常利用。

![pic2](https://pic.imgdb.cn/item/6536404bc458853aef4818d7.jpg)

依次将利用脚本拷贝到本地，分别检查的时候发现 Wolfcms 的后台登录界面：`http://[URL]/wolfcms/?/admin/login` , 打开后使用弱口令 `admin:admin` 成功登录后台

![pic3](https://pic.imgdb.cn/item/65363f78c458853aef465d48.jpg)

### 获取机器权限

浏览后台界面，发现在Snippets和Layouts处都可以写入 PHP 代码，因此直接写入大马反弹shell。此处使用的是 [revshells.com](https://www.revshells.com/) 自动生成的 PHP 大马。然后再打开插入了木马的文章，成功反弹shell。

![pic4](https://pic.imgdb.cn/item/65378393c458853aef7a311f.jpg)

回连的信息如下：

```
$ nc -lvnp 10234      
listening on [any] 10234 ...
connect to [192.168.213.129] from (UNKNOWN) [192.168.213.130] 49343
Linux SickOs 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 athlon i386 GNU/Linux
 18:09:34 up  6:48,  0 users,  load average: 0.00, 0.01, 0.03
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: no job control in this shell
www-data@SickOs:/$ 
```

## 提权


## 注释

### Squid 服务

>**Squid** is a caching and forwarding HTTP web proxy. It has a wide variety of uses, including speeding up a web server by caching repeated requests, caching web, DNS and other computer network lookups for a group of people sharing network resources, and aiding security by filtering traffic. Although primarily used for HTTP and FTP, Squid includes limited support for several other protocols including Internet Gopher, SSL, TLS and HTTPS. Squid does not support the SOCKS protocol, unlike Privoxy, with which Squid can be used in order to provide SOCKS support. (From [here](https://en.wikipedia.org/wiki/Squid_(software))).

Squid Cache（简称为 Squid ）是 HTTP 代理服务器软件。Squid 用途广泛，可以作为缓存服务器，可以过滤流量帮助网络安全，也可以作为代理服务器链中的一环，向上级代理转发数据或直接连接互联网。Squid 程序在 Unix 一类系统运行。由于它是开源软件，有网站修改 Squid 的源代码，编译为原生 Windows 版；用户也可在Windows 里安装 Cygwin ，然后在 Cygwin 里编译 Squid 。  
Squid 历史悠久，功能完善。除了 HTTP 外，对 FTP 与 HTTPS 的支持也相当好，在3.0 测试版中也支持了 IPv6 。但是 Squid 的上级代理不能使用 SOCKS 协议。