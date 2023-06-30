---
title: mkdocs构建自己的漏洞&PoC库
date: 2023-06-03 10:53:50
tags:
- mkdocs
- 漏洞&PoC库
categories: 漏洞&PoC库
keywords: 漏洞&PoC库
description: 收集了不少已经披露的漏洞和PoC，但是想自己挨个复现做个汇总，因此就有了构建自己的漏洞&PoC库这个想法。期间采用过 gitbook、honkit，但都效果不佳。兜兜转转最终选择使用mkdocs来构建整个文档。
top_img: https://pic.imgdb.cn/item/647c13abf024cca173275227.png
cover: https://pic.imgdb.cn/item/647c13abf024cca173275227.png
password: xdxdsqyc15511551
message: 暂未完成，仍在施工
---

## 背景条件

### 出发点

平时刷漏洞总感觉很细碎，总感觉没有形成一个规范化的学习过程，更别谈有一个成形的方法论了。而没有方法论，终究只是零碎的知识点，缺少一条最为关键的贯通线串联起来，因此运用也很吃力。

看到大佬直接用 0day 挖项目，十分震惊，更羡慕他的高效率，因此决定自己建立一个漏洞库。目前漏洞库的主要来源是 Awesome-PoC 这个仓库，后续可能会持续关注 Exploit-db 、Vulners 等漏洞库的最新情报。

## 二度踩坑

在我的设想里面，漏洞库应该使用许多md文档构建的，这样既方便书写，也方便整理。
一开始觉得 gitbook 用来写 Wiki 的界面很美观，而且经常也遇见类似的界面

## 最终结果