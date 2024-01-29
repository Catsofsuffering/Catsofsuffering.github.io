---
title: Splunk搭建SIEM
tags:
  - Splunk
  - SIEM
categories: SOC
keywords: Splunk
description: 本文记录了作者在使用Splunk搭建SIEM时所需的知识点以及操作流程
top_img: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
cover: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
---
## Splunk 简介

### 总览

Splunk 是一款强大的数据分析和监控平台，专注于处理大规模、复杂的数据集。它能够实时、高效地索引、搜索、可视化分析结构化和非结构化数据，为用户提供洞察业务、发现异常、解决问题的能力。
Splunk 支持从各种数据源中汇集数据，包括应用程序日志、网络流量、传感器数据等。用户可以通过简单直观的搜索查询语言，对数据进行高级搜索、分析和可视化。其强大的查询和分析功能，使得用户能够发现隐藏在数据中的关键信息、模式和趋势，用以优化业务运营、增强安全性和做出数据驱动的决策。
Splunk 还具备灵活的部署选项，可以在本地、云端或混合环境中部署，以满足不同组织的需求。综而言之，Splunk 是一个多功能、易用且高度可定制的数据分析平台，为企业提供了丰富的数据洞察，助力业务持续优化和创新。

### Splunk UF

Splunk Universal Forwarder（UF）是 Splunk 平台的一个重要组件，用于将数据从数据源（如服务器、终端设备、应用程序等）采集、转发并发送至 Splunk 索引器（Splunk Indexer）。UF 轻量且高效，设计用于在采集端点上安装和运行，以最小化资源占用和网络流量。

UF的主要功能包括：数据采集、日志转发、安全性、数据压缩、数据处理和数据路由。它能够采集多种数据类型，包括日志文件、事件数据、指标和其他格式的数据。一旦数据被采集，UF 会将其压缩并安全地发送到 Splunk 索引器进行索引、搜索和分析。

通过使用 Splunk UF ，组织可以实现实时数据采集，确保数据安全传输，降低网络流量，提高系统和网络性能。它允许企业集中管理数据采集配置，确保数据的完整性、可靠性和保密性。Splunk UF 是构建在 Splunk 平台上的重要工具，为实时数据采集和分析提供了可靠的基础。

### Splunk 下载

如果你是第一次访问 Splunk 网站，需要先注册一个 Splunk 用户，默认下载的是 60 天 Enterprise 试用版， 60 天试用之后将自动转化为 Free 版，转化位 Free 版后每日处理的日志量最高位 500M 。

下载地址: [Splunk Enterprise 9.1.1 下载](https://www.splunk.com/en_us/download/splunk-enterprise.html#) [Splunk Forwarder 9.1.1 下载](https://www.splunk.com/en_us/download/universal-forwarder.html)
## Splunk Server搭建

下载 Splunk 安装包之后，直接解压压缩包到指定目录安装即可，默认为 `/opt/splunk` 目录。同时，Splunk需要至少大于5GB的存储，所以最好在服务搭建的时候就设计好存储空间，以防后续数据过多影响Splunk的正常使用。
### 虚拟机添加硬盘

此处我使用的工具为 VMworkstation 17 Pro ，虚拟机环境为 Ubuntu 22.02 。

关于如何创建虚拟机，在此就不一一赘述。创建玩虚拟机之后，点击 `Edit virtual machine settings` ，添加硬盘/ `hard disk` ，如下图所示。然后使用默认推荐的种类即可，也就是 SCSI 硬盘，选择单个文件

![add another disk](https://pic.imgdb.cn/item/657fbf14c458853aefc303c1.jpg)

![recommended](https://pic.imgdb.cn/item/657fc0c8c458853aefc69fe1.jpg)

![create a new virtual disk](https://pic.imgdb.cn/item/657fc152c458853aefc7d4cb.jpg)
### 预先分配存储空间

首先使用 `fdisk` 命令查看分区信息<sup>[①](#理解-Linux-中的-fdisk-命令)</sup>。

```
fdisk -l
fdisk /dev/sdb
```

然后创建对应的物理卷、逻辑组和逻辑卷，并挂载到 `/opt/splunk` 安装目录下。创建服务账号 `splunk` ，将安装压缩包解压到 `/opt` 目录下（安装包中自带splunk文件夹），并将文件所属改为对应的服务账号，使用服务账号启动程序。
 
```
pvcreate /dev/sdb 
vgcreate splunkvg /dev/sdb
lvcreate -l 100%FREE -n splunklv splunkvg
mkfs.ext4 /dev/splunkvg/splunklv

mkdir -p /opt/splunk
mount /dev/splunkvg/splunklv /opt/splunk

cat << EOF >> /etc/fstab
/dev/mapper/splunkvg-splunklv /opt/splunk/ ext4 defaults 0 0
EOF

ulimit -n 65535
ulimit -u 20480

cat  << EOF >> /etc/security/limits.conf 
root       soft    nofile    65535
root       hard    nofile    65536
splunk     soft    nofile    65535
splunk     hard    nofile    65536
root       soft    nproc     20480
root       hard    nproc     20480
splunk     soft    nproc     20480
splunk     hard    nproc     20480
EOF

tar -xzvf /tmp/splunk-9.0.6-050c9bca8588-Linux-x86_64.tgz -C /opt/

groupadd splunk
useradd -m -r splunk -s /bin/bash -g splunk
chown -R splunk:splunk /opt/splunk/
su - splunk -c "/opt/splunk/bin/splunk start --accept-license --answer-yes"
```

注意提前备份 `/etc/fstab` 文件：

``` sh
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-S03D4YvWdcDtf5IZK3B0e1KQNjxajra7J7jqyif9xEbBZOeR08oTyCF1a2Lx90Qu / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/2c5535ce-cd76-4e93-9c37-c0ff0d5c87e9 /boot ext4 defaults 0 1
# Splunk
/dev/mapper/splunkvg-splunklv /opt/splunk/ ext4 defaults 0 0
# Gitlab
/dev/mapper/gitlabvg-gitlablv /opt/gitlab/ ext4 defaults 0 0
```
### 配置Splunk Security Essential

在 Splunk Web 界面上打开 `Manage Apps` 界面，直接点击 `Install app from file` 按钮，将 Splunkbase 上下载的安装包直接导入即可。

![Install Apps](https://pic.imgdb.cn/item/65ae2442871b83018abfadf8.jpg)
## 安装Gitlab

```
pvcreate /dev/sdc
vgcreate gitlabvg /dev/sdc
lvcreate -l 100%FREE -n gitlablv gitlabvg
mkfs.ext4 /dev/gitlabvg/gitlablv

mkdir -p /opt/gitlab
mount /dev/gitlabvg/gitlablv /opt/gitlab

cat << EOF >> /etc/fstab
/dev/mapper/gitlabvg-gitlablv /opt/gitlab/ ext4 defaults 0 0
EOF

mv /tmp/gitlab-ce_16.7.0-ce.0_amd64.deb /opt/gitlab/
groupadd gitlab
useradd -m -r gitlab -s /bin/bash -g root -G sudo
cd /opt/gitlab & dpkg -i gitlab-ce_16.7.0-ce.0_amd64.deb
chown -R gitlab:root /opt/gitlab/

su - gitlab -c "cp /opt/gitlab/etc/gitlab.rb.template /opt/gitlab/etc/gitlab.rb"

su - gitlab -c "sed -i 's|external_url '\''GENERATED_EXTERNAL_URL'\''|external_url '\''http://localhost:9999'\''|' /opt/gitlab/etc/gitlab.rb"

```
## Splunk Forwarder 安装与配置

下载 Splunk Forwarder 安装包之后，将安装包在目标服务器中解压，此处作者选择在平时直接解压压缩包到指定目录安装即可，默认为 `/opt/splunkforwarder` 目录。
## 注释

### 理解 Linux 中的 fdisk 命令

在 Linux 系统中，`fdisk` 是一个用于处理磁盘分区的关键命令。通过提供一个交互式的命令行界面，`fdisk` 允许用户对硬盘进行创建、删除、调整和查看分区的操作。下面是一些 `fdisk` 命令的基本用法：

1. **查看磁盘分区信息**：

   ```bash
   fdisk -l
   ```

   这个命令将列出系统上所有磁盘的分区信息，包括磁盘的大小、分区类型等。

2. **打开一个磁盘进行分区**：

   ```bash
   fdisk /dev/sdX
   ```

   这里的 `/dev/sdX` 代表要进行分区操作的磁盘，例如 `/dev/sda`。

3. **显示分区列表**：

   在 `fdisk` 的交互式界面中，输入 `p` 可以显示磁盘的分区列表，包括分区的起始扇区和大小。

4. **创建新的分区**：

   在 `fdisk` 的交互式界面中，输入 `n` 可以启动创建新分区的过程，用户需要提供分区的类型、起始扇区和大小。

5. **删除分区**：

   在 `fdisk` 的交互式界面中，输入 `d` 可以删除已有分区。用户需要确认删除的分区编号。

6. **保存更改**：

   在 `fdisk` 的交互式界面中，完成分区操作后，输入 `w` 可以保存所做的更改。请谨慎使用此命令，因为它会直接修改磁盘上的分区表。

7. **退出 `fdisk`**：

   在 `fdisk` 的交互式界面中，输入 `q` 可以退出而不保存更改，而输入 `w` 可以退出并保存更改。

这仅仅是 `fdisk` 命令的基本用法。在进行实际的硬盘操作时，需要谨慎使用，确保提前备份好重要数据。分区操作可能会对数据造成不可逆的影响，特别是在不熟悉分区概念和操作的情况下。