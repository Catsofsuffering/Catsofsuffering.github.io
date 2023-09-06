---
title: idea使用踩坑
tags:
---

## idea JVM启动失败



## idea 控制台中文乱码

重新安装了idea后发现控制台日志中文乱码

1、修改IDEA配置文件

IDEA安装目录的bin文件夹下 `idea64.exe.vmoptions` 和 `idea.exe.vmoptions` 分别添加：``-Dfile.encoding=UTF-8`。

2、修改IDEA设置

FileEncoding 中的 GlobalEncoding ProjectEncoding Default encoding for properties 都设置为UTF-8。

3、在部署服务器的VM options中添加：`-Dfile.encoding=UTF-8`

4、重启IDEA(一定要重启)

 

这次调整只使用了  1、2、4 三个步骤，控制台日志中文乱码解决