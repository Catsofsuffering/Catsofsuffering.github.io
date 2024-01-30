---
title: Splunk自定义威胁情报命令
date: 2024-01-18 15:34:35
tags:
  - SOC
  - Splunk
categories: 安全运营
description: 本文记录了作者在开发Splunk自定义命令，通过调用API拉取微步情报信息，所需的知识点以及操作流程
top_img: https://pic.imgdb.cn/item/65b860e0871b83018a7e5d20.jpg
cover: https://pic.imgdb.cn/item/65b860e0871b83018a7e5d20.jpg
---
## 事前准备
### 配置 VSCode 调试插件

分别在 `Splunk` 和 `VSCode` 上安装对应的插件：[Splunk Add-on for Microsoft Visual Studio Code](https://splunkbase.splunk.com/app/4801) 和 [Splunk Extension](https://marketplace.visualstudio.com/items?itemName=Splunk.splunk)

在VSCode的settings.json中配置Splunk：

``` json
"splunk.spec.FilePath":  <$SPLUNK_HOME/etc/system/README/>,
// "splunk.spec.FilePath": "/opt/splunk/etc/system/README/"
"splunk.commands.token": <token>,
// "splunk.commands.token": "eyJraWQiOiJzcGx1bmsuc2VjcmV0IiwiYWxnIjoiSFM1MTIiLCJ2ZXIiOiJ2MiIsInR0eXAiOiJzdGF0aWMifQ.eyJpc3MiOiJwZW9ueSBmcm9tIHRlc3Rtb25pdG9yIiwic3ViIjoicGVvbnkiLCJhdWQiOiJWU0MiLCJpZHAiOiJTcGx1bmsiLCJqdGkiOiJiODg4ODYxNjljYWQyZmU1OWFlNGMwY2U5N2NlMDE5NmE3OTBiNDY1YWVkMjA1OGUyMDA3NzU1ZTNiZTg1ZWNlIiwiaWF0IjoxNzA1Mjg0MjM3LCJleHAiOjE3MDc4NzYyMzcsIm5iciI6MTcwNTI4NDIzN30.I6JGbcxGIJLSoqhMVG7acdAH7C4xCT_iZBfe1JQAccAuvKgxLjsdbAnw5w5vzT5793R8X8yPeS-avYU7wMilWA",
```

token 获取方法（Splunk系统必须启用KVStore）：

![token](https://pic.imgdb.cn/item/65a8d776871b83018a0222a2.jpg)

### 配置 Splunk 调试插件

在 Splunk Web 界面上打开 `Manage Apps` 界面，直接点击 `Install app from file` 按钮，将 Splunkbase 上下载的安装包直接导入即可。

![Install Apps](https://pic.imgdb.cn/item/65ae2442871b83018abfadf8.jpg)

安装完成之后，可以看到 `Apps` 中已经存在  `Microsoft Visual Studio Code Supporting Add-on for Splunk` 这个App，打开 App 可以看到调试方法。

![VSC Ad-ons on Splunk](https://pic.imgdb.cn/item/65ae26cc871b83018ac911fe.jpg)
### 使用自定义脚本模板开发

打开 `VSCode` 命令面板（`Ctrl+Shift+P`），输入 `Splunk` ，找到VSC插件内置好的脚手架：

![scaffold](https://pic.imgdb.cn/item/65ae0eff871b83018a74be63.jpg)

### 下载Splunk-SDK

使用下方命令将 `splunk-sdk` 安装到自定义Apps的 `\bin` 目录下<sup>[①](#pip-install-参数解释)</sup>：

```
pip install -t $SPLUNK_HOME/etc/apps/<APP_NAME>/bin splunk-sdk
```

同理，安装其他第三方库也可以用同样的命令：

```
pip install -t $SPLUNK_HOME/etc/apps/<APP_NAME>/bin <install-pip-package>
```

## 编写脚本

Github传送门： [xThreatBook](https://github.com/Catsofsuffering/xThreatBook.git)
## 执行流程

![the cummand prosess](https://pic.imgdb.cn/item/65aa0e17871b83018aa95570.jpg)

## 添加提示信息

根据[官方文档](https://docs.splunk.com/Documentation/Splunk/9.1.2/Admin/Searchbnfconf)，编写如下提示信息：

``` ini
[threatinfo-command]

syntax      = threatinfo threat_url=<fieldname> (query_type=(scene_ip_reputation))?  (query_type=(ip_query))?  (query_type=(domain_query))? (query_type=(scene_dns))? (query_type=(ip_adv_query))? (query_type=(domain_adv_query))? (query_type=(domain_sub_domains))? (query_type=(scene_domain_context))?

shortdesc   = Fetch threat information from XThreatBook.

description = This command could fetch threat information from XThreatBook by API according to the IPs or urls which the user put in.\i\

    <query_type> - Optional, determine which type of XThreatBook API to use. \i\

    <threat_url> - The "url" must be a field, and the result will be output into field which name "response".

example1 = .. | threatinfo threat_url=url

comment1 = The "url" must be a field, and the result will be output into field which name "response". Then you can use SPL command "spath" to parse the result.

related = spath

usage = public

tags = threat_url query_type

maintainer = Ventus
```
## 注释

### pip install 参数解释

使用 `pip help install` 命令可以看到对应解释：

```
  -t, --target <dir>          Install packages into <dir>. By default this
                              will not replace existing files/folders in
                              <dir>. Use --upgrade to replace existing
                              packages in <dir> with new versions.
```