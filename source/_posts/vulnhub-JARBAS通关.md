---
title: vulnhub-JARBAS通关
date: 2023-06-10 10:00:24
tags:
- vulnhub
- jarbas
description: 本文记录了作者在通关vulnhub靶机JARBAS中遇到的问题以及踩坑后的解决方案。（摘要补充）
top_img: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
cover: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
password: xdxdsqyc15511551
message: 暂未完成，仍在施工
---


## 信息搜集

靶机地址：192.168.118.135

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]
└─$ sudo nmap -sT -p- --min-rate 10000 192.168.118.135 -oN nmapscan/portscan.txt    
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-09 23:24 EDT
Nmap scan report for 192.168.118.135
Host is up (0.0013s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:4C:CC:C8 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 3.52 seconds
```

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]
└─$ sudo nmap -sT -Pn -sCV -O -p22,80,3306,8080 --min-rate 10000 192.168.118.135 -oN nmapscan/servscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-09 23:30 EDT
Nmap scan report for 192.168.118.135
Host is up (0.00065s latency).

PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 28bc493c6c4329573cb8859a6d3c163f (RSA)
|   256 a01b902cda79eb8f3b14debb3fd2e73f (ECDSA)
|_  256 57720854b756ffc3e6166f97cfae7f76 (ED25519)
80/tcp   closed http
3306/tcp open   mysql      MariaDB (unauthorized)
8080/tcp closed http-proxy
MAC Address: 00:0C:29:4C:CC:C8 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.54 seconds
```

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]
└─$ sudo nmap -sU -Pn -p- --min-rate 10000 192.168.118.135 -oN nmapscan/udpscan.txt                                                              
[sudo] kali 的密码：
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-09 23:31 EDT
Warning: 192.168.118.135 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.118.135
Host is up (0.0014s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT      STATE SERVICE
33848/udp open  unknown
MAC Address: 00:0C:29:4C:CC:C8 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 72.82 seconds
```

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]                                [172/189]
└─$ dirsearch -u http://192.168.118.135                                                    
                         
  _|. _ _  _  _  _ _|_    v0.4.2                                                           
 (_||| _) (/_(_|| (_| )  
                         
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30                      
Wordlist size: 10927     
                         
Output File: /home/kali/.dirsearch/reports/192.168.118.135/_23-06-09_23-32-46.txt                                                                 
             
Error Log: /home/kali/.dirsearch/logs/errors-23-06-09_23-32-46.log             
             
Target: http://192.168.118.135/                                                
             
[23:32:46] Starting:                                                           
[23:32:48] 403 -  213B  - /.ht_wsr.txt                                         
[23:32:48] 403 -  218B  - /.htaccess.sample                                    
[23:32:48] 403 -  216B  - /.htaccess.bak1                                      
[23:32:48] 403 -  216B  - /.htaccess.orig                                      
[23:32:48] 403 -  217B  - /.htaccess_extra                                     
[23:32:48] 403 -  214B  - /.htaccess_sc                                        
[23:32:48] 403 -  216B  - /.htaccess_orig                                      
[23:32:48] 403 -  214B  - /.htaccessBAK                                        
[23:32:48] 403 -  215B  - /.htaccessOLD2                                     
[23:32:48] 403 -  216B  - /.htaccess.save                                     
[23:32:48] 403 -  214B  - /.htaccessOLD                                       
[23:32:48] 403 -  206B  - /.htm        
[23:32:48] 403 -  207B  - /.html       
[23:32:48] 403 -  216B  - /.htpasswd_test                                     
[23:32:48] 403 -  213B  - /.httr-oauth                                        
[23:32:48] 403 -  212B  - /.htpasswds                                         
[23:32:55] 200 -  359B  - /access.html                                        
[23:33:03] 403 -  210B  - /cgi-bin/                                           
[23:33:11] 200 -   32KB - /index.html                                         

Task Completed 
```



robots.txt
```
# we don't want robots to click "build" links
User-agent: *
Disallow: /
```

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]      
└─$ hash-identifier 5978a63b4654c73c60fa24f836386d87    
   #########################################################################                                                           
   #     __  __                     __           ______    _____           #                                                           
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #                                                           
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #                                                           
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #                                                           
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #                                                           
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #                                                           
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #                                                           
   #                                                             By Zion3R #                                                           
   #                                                    www.Blackploit.com #                                                           
   #                                                   Root@Blackploit.com #                                                           
   #########################################################################                                                           
--------------------------------------------------      
                                                        
Possible Hashs:                                         
[+] MD5                                                 
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))  
```

```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]
└─$ cat user_passwd.txt 
tiago:italia99
trindade:marianna
eder:vipsu
```

前两个登录8080端口后台都失败了，我以为这条路又走不通，浪费了半小时试了最后一个，唉。



```
┌──(kali㉿kali)-[~/Desktop/BoxWalk/Vulnhub/jarbas]
└─$ dirsearch -u http://192.168.118.136:8080 --cookie="JSESSIONID.d771878c=node08k4qeinw3ejxq54z5xgx0p1012683.node0; ACEGI_SECURITY_HASHED_REMEMBER_ME_COOKIE=ZWRlcjoxNjg3NjI4MDc5Mjg1OmQxYzMwYzkwOGE0M2ExM2FhMjhmYTcyODlmOTdkYWNhZGVjMWU4ZTEyMzRlYmJjMmY0ZmQyYzJkZGYyYmFmNmQ=" -x 403


  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30
Wordlist size: 10927

Output File: /home/kali/.dirsearch/reports/192.168.118.136-8080/_23-06-11_03-26-51.txt

Error Log: /home/kali/.dirsearch/logs/errors-23-06-11_03-26-51.log

Target: http://192.168.118.136:8080/

[03:26:52] Starting: 
[03:27:00] 302 -    0B  - /about  ->  http://192.168.118.136:8080/about/
[03:27:00] 302 -    0B  - /actions  ->  http://192.168.118.136:8080/actions/
[03:27:06] 302 -    0B  - /api  ->  http://192.168.118.136:8080/api/
[03:27:07] 302 -    0B  - /assets  ->  http://192.168.118.136:8080/assets/
[03:27:07] 500 -   13KB - /assets/
[03:27:07] 302 -    0B  - /authentication  ->  http://192.168.118.136:8080/authentication/
[03:27:07] 200 -   13KB - /api/
[03:27:09] 302 -    0B  - /class  ->  http://192.168.118.136:8080/class/
[03:27:09] 302 -    0B  - /columns  ->  http://192.168.118.136:8080/columns/
[03:27:10] 200 -   22KB - /cli/
[03:27:10] 200 -  237B  - /config.xml
[03:27:10] 303 -    0B  - /console/j_security_check  ->  http://192.168.118.136:8080/loginError
[03:27:11] 302 -    0B  - /credentials  ->  http://192.168.118.136:8080/credentials/
[03:27:12] 200 -   13KB - /credentials/
[03:27:14] 400 -    6KB - /error
[03:27:15] 200 -   17KB - /favicon.ico
[03:27:20] 200 -   11KB - /index
[03:27:20] 200 -   11KB - /instance/
[03:27:20] 303 -    0B  - /j_security_check  ->  http://192.168.118.136:8080/loginError
[03:27:23] 302 -    0B  - /log  ->  http://192.168.118.136:8080/log/
[03:27:23] 200 -    9KB - /log/
[03:27:23] 200 -   11KB - /login
[03:27:24] 302 -    0B  - /logout  ->  http://192.168.118.136:8080/
[03:27:24] 302 -    0B  - /logout/  ->  http://192.168.118.136:8080/
[03:27:27] 500 -   13KB - /main
[03:27:33] 200 -   17KB - /manage
[03:27:34] 302 -    0B  - /nodes  ->  http://192.168.118.136:8080/nodes/
[03:27:39] 302 -    0B  - /people  ->  http://192.168.118.136:8080/people/
[03:27:45] 302 -    0B  - /projects  ->  http://192.168.118.136:8080/projects/
[03:27:46] 302 -    0B  - /properties  ->  http://192.168.118.136:8080/properties/
[03:27:49] 302 -    0B  - /root  ->  http://192.168.118.136:8080/root/
[03:27:49] 200 -   71B  - /robots.txt
[03:27:51] 302 -    0B  - /search  ->  http://192.168.118.136:8080/search/
[03:27:51] 401 -    0B  - /secured
[03:27:51] 200 -   11KB - /restart
[03:27:51] 302 -    0B  - /security  ->  http://192.168.118.136:8080/security/
[03:27:53] 200 -   12KB - /script/jqueryplugins/dataTables/extras/TableTools/media/swf/ZeroClipboard.swf
[03:27:53] 200 -   12KB - /script/
[03:27:53] 200 -   12KB - /script
[03:28:00] 302 -    0B  - /target  ->  http://192.168.118.136:8080/target/
[03:28:05] 302 -    0B  - /url  ->  http://192.168.118.136:8080/url/
[03:28:06] 200 -   11KB - /target/                                         
[03:28:07] 302 -    0B  - /version  ->  http://192.168.118.136:8080/version/
[03:28:07] 302 -    0B  - /views  ->  http://192.168.118.136:8080/views/   
                                                                           
Task Completed
```