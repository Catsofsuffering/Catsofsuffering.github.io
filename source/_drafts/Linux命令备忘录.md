---
title: Linux命令备忘录
tags: 备忘录
---

# Linux命令备忘录

`ls`命令：用于列出当前目录下的文件和文件夹。可以使用不同的选项来显示不同的信息。例如，`ls -l`可以显示文件的详细信息，而`ls -a`可以显示所有文件和文件夹，包括隐藏文件和文件夹。

`cd`命令：用于切换当前工作目录。可以使用绝对路径或相对路径来指定目录。例如，`cd /usr/bin`将当前工作目录更改为`/usr/bin`目录，而`cd ..`将当前工作目录更改为上一级目录。

`pwd`命令：用于显示当前工作目录的路径。

`mkdir`命令：用于创建一个新的目录。例如，`mkdir test`将创建一个名为`test`的新目录。

`rmdir`命令：用于删除一个空目录。例如，`rmdir test`将删除名为`test`的空目录。

`cp`命令：用于复制文件或目录。例如，`cp file1 file2`将文件`file1`复制到文件`file2`中，而`cp -r dir1 dir2`将目录`dir1`复制到目录`dir2`中。

`rm`命令：用于删除文件或目录。例如，`rm file1`将删除名为`file1`的文件，而`rm -r dir1`将删除名为`dir1`的目录及其所有内容。

`mv`命令：用于移动或重命名文件或目录。例如，`mv file1 file2`将文件`file1`重命名为`file2`，而`mv file1 dir1`将文件`file1`移动到目录`dir1`中。

`cat`命令：用于查看文件内容。例如，`cat file1`将显示文件`file1`的内容。

`touch`命令：用于创建一个空文件或更改文件的修改时间戳。例如，`touch file1`将创建一个名为`file1`的空文件，而`touch -m file1`将修改文件file1的修改时间戳。

`chmod`命令：用于更改文件或目录的权限。例如，`chmod 755 file1`将文件file1的权限设置为`755`。

`chown`命令：用于更改文件或目录的所有者。例如，`chown user1 file1`将文件`file1`的所有者更改为`user1`。

`ps`命令：用于显示当前正在运行的进程。例如，`ps aux`将显示所有正在运行的进程的详细信息。

`kill`命令：用于终止一个进程。例如，`kill 1234`将终止进程ID为1234的进程。

`ping`命令：用于测试与另一个计算机的连接状态。例如，`ping google.com`将测试与Google服务器的连接状态。

`top`命令：用于显示当前系统资源的使用情况。例如，`top`将显示CPU、内存和进程等系统资源的使用情况。

`ifconfig`命令：用于显示网络接口配置信息。例如，ifconfig将显示当前系统的网络接口配置信息。

`netstat`命令：用于显示网络连接状态和统计信息

`df`命令：用于显示磁盘空间使用情况。例如，`df -h`将以人类可读的方式显示磁盘空间使用情况。

`du`命令：用于显示目录或文件的磁盘使用情况。例如，`du -h dir1`将以人类可读的方式显示目录dir1的磁盘使用情况。

`tar`命令：用于压缩和解压缩文件和目录。例如，`tar -czvf archive.tar.gz dir1`将目录dir1压缩为一个名为archive.tar.gz的文件，而`tar -xzvf archive.tar.gz`将解压缩名为archive.tar.gz的文件。

`grep`命令：用于在文件中查找匹配的文本。例如，`grep "pattern" file1`将在文件file1中查找匹配模式的文本。

`find`命令：用于在文件系统中查找文件和目录。例如，`find / -name "file1"`将在整个文件系统中查找名为file1的文件。

`ssh`命令：用于连接到远程服务器。例如，`ssh user1@remotehost.com`将使用用户名user1连接到名为remotehost.com的远程服务器。

`scp`命令：用于在本地计算机和远程服务器之间复制文件。例如，`scp file1 user1@remotehost.com:/home/user1`将文件`file1`复制到名为`remotehost.com`的远程服务器上的/home/user1目录中。

`wget`命令：用于从网络上下载文件。例如，`wget http://www.example.com/file1`将从名为www.example.com的网站上下载文件file1。

`curl`命令：用于从网络上获取和发送数据。例如，`curl http://www.example.com`将获取名为www.example.com的网站的数据。

`uname`命令：用于显示当前操作系统的信息。例如，`uname -a`将显示完整的操作系统信息。
