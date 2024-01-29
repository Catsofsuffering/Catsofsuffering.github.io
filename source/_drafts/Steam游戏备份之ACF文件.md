---
title: Steam游戏备份之ACF文件
tags:
---
## ACF 文件

ACF 文件是 Steam Application Cache File 的缩写

``` acf
"AppState"

{

    "appid""206440"（游戏的id，重要，你可以根据游戏名称查到）
    "Universe""1""name""To the Moon"（游戏名称，同时也是目录名字）
    "StateFlags""4"（这里有两个值：“1024“代表未下载完，“4”代表安装完成）
    "installdir""To the Moon""LastUpdated""1430538160"（和更新相关的）
    "UpdateResult""0"（“0”代表无需更新）
    "SizeOnDisk""108715461"（磁盘占用空间）
    "buildid""588273"（很熟悉吧）
    "LastOwner""7656*********9065"（购买人的id）
    "BytesToDownload""95069216"（下载大小）
    "BytesDownloaded""95069216"（已下载大小）
    "AutoUpdateBehavior""0""AllowOtherDownloadsWhileRunning""0""UserConfig"{
        "language""schinese"
    }
    "MountedDepots"（这个比较重要，你需要在，点击进入。里面查找相关的数据，https: //steamdb.info/app/***/，*里面填写appid可以找到相关游戏信息，https://steamdb.info/app/***/depots/，里面可以找到下面两行左边的数字，可能不止两个，https://steamdb.info/depot/***/把下面左边的数子写到*里面能查到右边的数字）
    {
        "206441""6387052800794306865""206454""467587817757935686"
    }
}
```