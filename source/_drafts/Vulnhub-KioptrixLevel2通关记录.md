---
title: Vulnhub-KioptrixLevel2通关记录
tags:
  - Kioptrix
  - vulnhub
  - WriteUp
categories: WriteUp
keywords: vulnhub
description: 本文记录了作者在通关vulnhub靶机JARBAS中遇到的问题以及踩坑后的解决方案。
top_img: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
cover: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
date: 2023-10-30 17:09:20
---

## 靶机描述

靶机地址：10.8.12.130

## 信息收集

### nmap扫描

按照常规的方法：通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息

下面是通过 TCP 扫描发现所收集到的信息：

```
$ sudo nmap -sT -p- --min-rate 10000 10.8.12.130 -oN scanoutput/portscan.txt

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
631/tcp  open  ipp
749/tcp  open  kerberos-adm
3306/tcp open  mysql
```


```
$ sudo nmap -sT -p22,80,111,443,631,821,3306 -sV --min-rate 10000 192.168.213.133 -oN nmapscan/servscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-11-09 04:28 EST
Nmap scan report for 192.168.213.133
Host is up (0.00097s latency).

PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open   http     Apache httpd 2.0.52 ((CentOS))
111/tcp  open   rpcbind  2 (RPC #100000)
443/tcp  open   ssl/http Apache httpd 2.0.52 ((CentOS))
631/tcp  open   ipp      CUPS 1.1
821/tcp  closed unknown
3306/tcp open   mysql    MySQL (unauthorized)
MAC Address: 00:0C:29:53:19:56 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.56 seconds
```
### Web 目录扫描

```
$ cat scanoutput/nikto_scan.txt
- Nikto v2.5.0/
+ Target Host: 10.8.12.130
+ Target Port: 80
+ GET /: Retrieved x-powered-by header: PHP/4.3.9.
+ GET /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options:
+ GET /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/:
+ HEAD Apache/2.0.52 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ OPTIONS OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE .
+ XPTDBYEG /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ TRACE /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing:
+ GET /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184:
+ GET /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184:
+ GET /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184:
+ GET /manual/: Uncommon header 'tcn' found, with contents: choice.
+ GET /manual/: Web server manual found.
+ GET /icons/: Directory indexing found.
+ GET /manual/images/: Directory indexing found.
+ GET /icons/README: Server may leak inodes via ETags, header found with file /icons/README, inode: 357810, size: 4872, mtime: Sun Mar 30 02:41:04 1980. See: CVE-2003-1418:
+ GET /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/:
+ GET /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
```

## 获取立足点

## 注释

