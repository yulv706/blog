---
{"dg-publish":true,"title":"hackmyvm_Slackware","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_Slackware.md","permalink":"/安全/靶机/hackmyvm_Slackware/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~]
└─# arp-scan --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:ac:e0:91, IPv4: 192.168.3.3
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.3.1     00:50:56:c0:00:08       VMware, Inc.
192.168.3.2     00:50:56:ef:95:8d       VMware, Inc.
192.168.3.4     08:00:27:ec:6b:21       PCS Systemtechnik GmbH
192.168.3.254   00:50:56:f8:1e:fc       VMware, Inc.

6 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.301 seconds (111.26 hosts/sec). 4 responded

┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 192.168.3.4
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 01:17 EST
Nmap scan report for 192.168.3.4
Host is up (0.029s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT  STATE SERVICE
1/tcp open  tcpmux
2/tcp open  compressnet
MAC Address: 08:00:27:EC:6B:21 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 12.82 seconds
```

开了两个端口，但是怎么这么奇怪，详细扫扫看看
```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# nmap -sT -sC -sV -O -p1,2 192.168.3.4
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 01:39 EST
Nmap scan report for 192.168.3.4
Host is up (0.0024s latency).

PORT  STATE SERVICE VERSION
1/tcp open  ssh     OpenSSH 9.3 (protocol 2.0)
| ssh-hostkey:
|   256 e2:66:60:79:bc:d1:33:2e:c1:25:fa:99:e5:89:1e:d3 (ECDSA)
|_  256 98:59:c3:a8:2b:89:56:77:eb:72:4a:05:90:21:cb:40 (ED25519)
2/tcp open  http    Apache httpd 2.4.58 ((Unix))
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Tribute to Slackware
|_http-server-header: Apache/2.4.58 (Unix)
MAC Address: 08:00:27:EC:6B:21 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.79 seconds
```
好吧就是ssh和web



# 初步渗透

```sh
┌──(root㉿kali)-[~]
└─# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.3.4:2/ -x .php,.html,.zip,.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.3.4:2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,zip,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 199]
/index.html           (Status: 200) [Size: 7511]
/robots.txt           (Status: 200) [Size: 21]
/.html                (Status: 403) [Size: 199]
/getslack             (Status: 301) [Size: 238] [--> http://192.168.3.4:2/getslack/]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================

```

```sh
┌──(root㉿kali)-[~]
└─# curl http://192.168.3.4:2/robots.txt
User-agent: *
#7z.001
```
不知道有啥用

再简单扫一下子目录吧
```sh
┌──(root㉿kali)-[~]
└─# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.3.4:2//getslack/ -x .php,.html,.zip,.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.3.4:2//getslack/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,zip,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 199]
/index.html           (Status: 200) [Size: 12]
/.html                (Status: 403) [Size: 199]
Progress: 901417 / 1102805 (81.74%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 901742 / 1102805 (81.77%)
===============================================================
Finished
===============================================================
```


```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# curl http://192.168.3.4:2/getslack/index.html
search here
```
哦！提示


去看眼WP
```sh
wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://192.168.3.4:2//getslack/FUZZ.7z.001  --hc 404
```
可以出来这么一个结果
```sh
http://192.168.3.4:2//getslack/twitter.7z.001
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# file twitter.7z.001
twitter.7z.001: 7-zip archive data, version 0.4

┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# for i in $(seq 1 20); do wget http://192.168.3.4:2/getslack/twitter.7z.0$i; done
....
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# for i in $(seq 1 20); do wget http://192.168.3.4:2/getslack/twitter.7z.00$i; done
....

┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# ls
twitter.7z.001  twitter.7z.005  twitter.7z.009  twitter.7z.013
twitter.7z.002  twitter.7z.006  twitter.7z.010  twitter.7z.014
twitter.7z.003  twitter.7z.007  twitter.7z.011
twitter.7z.004  twitter.7z.008  twitter.7z.012

┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# 7z x twitter.7z.001

7-Zip 24.08 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-08-11
 64-bit locale=en_US.UTF-8 Threads:32 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 20480 bytes (20 KiB)

Extracting archive: twitter.7z.001
--
Path = twitter.7z.001
Type = Split
Physical Size = 20480
Volumes = 14
Total Physical Size = 268100
----
Path = twitter.7z
Size = 268100
--
Path = twitter.7z
Type = 7z
Physical Size = 268100
Headers Size = 130
Method = LZMA2:384k
Solid = -
Blocks = 1

Everything is Ok

Size:       267951
Compressed: 268100
```

解压出一个图片
```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# file twitter.png
twitter.png: PNG image data, 400 x 400, 8-bit/color RGB, non-interlaced

```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# exiftool twitter.png
ExifTool Version Number         : 13.00
File Name                       : twitter.png
Directory                       : .
File Size                       : 268 kB
File Modification Date/Time     : 2024:03:10 16:42:47-04:00
File Access Date/Time           : 2025:02:17 02:33:02-05:00
File Inode Change Date/Time     : 2025:02:17 02:32:16-05:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Profile Name                    : icc
Profile CMM Type                : Little CMS
Profile Version                 : 4.4.0
Profile Class                   : Display Device Profile
Color Space Data                : RGB
Profile Connection Space        : XYZ
Profile Date Time               : 2022:12:19 06:28:40
Profile File Signature          : acsp
Primary Platform                : Apple Computer Inc.
CMM Flags                       : Not Embedded, Independent
Device Manufacturer             :
Device Model                    :
Device Attributes               : Reflective, Glossy, Positive, Color
Rendering Intent                : Perceptual
Connection Space Illuminant     : 0.9642 1 0.82491
Profile Creator                 : Little CMS
Profile ID                      : 0
Profile Description             : GIMP built-in sRGB
Profile Copyright               : Public Domain
Media White Point               : 0.9642 1 0.82491
Chromatic Adaptation            : 1.04788 0.02292 -0.05022 0.02959 0.99048 -0.01707 -0.00925 0.01508 0.75168
Red Matrix Column               : 0.43604 0.22249 0.01392
Blue Matrix Column              : 0.14305 0.06061 0.71393
Green Matrix Column             : 0.38512 0.7169 0.09706
Red Tone Reproduction Curve     : (Binary data 32 bytes, use -b option to extract)
Green Tone Reproduction Curve   : (Binary data 32 bytes, use -b option to extract)
Blue Tone Reproduction Curve    : (Binary data 32 bytes, use -b option to extract)
Chromaticity Channels           : 3
Chromaticity Colorant           : Unknown
Chromaticity Channel 1          : 0.64 0.33002
Chromaticity Channel 2          : 0.3 0.60001
Chromaticity Channel 3          : 0.15001 0.06
Device Mfg Desc                 : GIMP
Device Model Desc               : sRGB
White Point X                   : 0.3127
White Point Y                   : 0.329
Red X                           : 0.64
Red Y                           : 0.33
Green X                         : 0.3
Green Y                         : 0.6
Blue X                          : 0.15
Blue Y                          : 0.06
Warning                         : [minor] Trailer data after PNG IEND chunk
Image Size                      : 400x400
Megapixels                      : 0.160
```
看到后面有提示，tail一下图片
```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# tail twitter.png

...乱码...

trYth1sPasS1993
```
看一下那个照片
![Pasted image 20250217200826.png](/img/user/picture/Pasted%20image%2020250217200826.png)
ssh用户名就是这个推特的名字


```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# ssh patrick@192.168.3.4 -p 1
The authenticity of host '[192.168.3.4]:1 ([192.168.3.4]:1)' can't be established.
ED25519 key fingerprint is SHA256:m/iaIzavXraumIPoCQReEwCgahrbGQe8WpPXO8nfAqE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.3.4]:1' (ED25519) to the list of known hosts.
(patrick@192.168.3.4) Password:
Linux 5.15.145.
patrick@slackware:~$ id
uid=1000(patrick) gid=1000(patrick) groups=1000(patrick),1001(kretinga)
```

# 提权
```sh
patrick@slackware:/home$ cat /etc/passwd
root:x:0:0::/root:/bin/bash
bin:x:1:1:bin:/bin:/bin/false
daemon:x:2:2:daemon:/sbin:/bin/false
adm:x:3:4:adm:/var/log:/bin/false
lp:x:4:7:lp:/var/spool/lpd:/bin/false
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/:/bin/false
news:x:9:13:news:/usr/lib/news:/bin/false
uucp:x:10:14:uucp:/var/spool/uucppublic:/bin/false
operator:x:11:0:operator:/root:/bin/bash
games:x:12:100:games:/usr/games:/bin/false
ftp:x:14:50::/home/ftp:/bin/false
smmsp:x:25:25:smmsp:/var/spool/clientmqueue:/bin/false
mysql:x:27:27:MySQL:/var/lib/mysql:/bin/false
rpc:x:32:32:RPC portmap user:/:/bin/false
sshd:x:33:33:sshd:/:/bin/false
gdm:x:42:42:GDM:/var/lib/gdm:/sbin/nologin
ntp:x:44:44:User for NTP:/:/bin/false
icecc:x:49:49:User for Icecream distributed compiler:/var/cache/icecream:/bin/false
oprofile:x:51:51:oprofile:/:/bin/false
usbmux:x:52:83:User for usbmux daemon:/var/empty:/bin/false
named:x:53:53:User for BIND:/var/named:/bin/false
sddm:x:64:64:User for SDDM:/var/lib/sddm:/bin/false
pulse:x:65:65:User for PulseAudio:/var/run/pulse:/bin/false
dhcpcd:x:68:68:User for dhcpcd:/var/lib/dhcpcd:/bin/false
apache:x:80:80:User for Apache:/srv/httpd:/bin/false
messagebus:x:81:81:User for D-BUS:/var/run/dbus:/bin/false
haldaemon:x:82:82:User for HAL:/var/run/hald:/bin/false
polkitd:x:87:87:PolicyKit daemon owner:/var/lib/polkit:/bin/false
pop:x:90:90:POP:/:/bin/false
postfix:x:91:91:User for Postfix MTA:/dev/null:/bin/false
dovecot:x:94:94:User for Dovecot processes:/dev/null:/bin/false
dovenull:x:95:95:User for Dovecot login processing:/dev/null:/bin/false
nobody:x:99:99:nobody:/:/bin/false
ldap:x:330:330:OpenLDAP server:/var/lib/openldap:/bin/false
patrick:x:1000:1000::/home/patrick:/bin/bash
kretinga:x:1001:1001::/home/kretinga:/bin/bash
claor:x:1002:1002::/home/claor:/bin/bash
alienum:x:1003:1003::/home/alienum:/bin/bash
mrmidnight:x:1004:1004::/home/mrmidnight:/bin/bash
annlynn:x:1005:1005::/home/annlynn:/bin/bash
powerful:x:1006:1006::/home/powerful:/bin/bash
proxy:x:1007:1007::/home/proxy:/bin/bash
x4v1l0k:x:1008:1008::/home/x4v1l0k:/bin/bash
icex64:x:1009:1009::/home/icex64:/bin/bash
mindsflee:x:1010:1010::/home/mindsflee:/bin/bash
zacarx007:x:1011:1011::/home/zacarx007:/bin/bash
terminal:x:1012:1012::/home/terminal:/bin/bash
zenmpi:x:1013:1013::/home/zenmpi:/bin/bash
sml:x:1014:1014::/home/sml:/bin/bash
emvee:x:1015:1015::/home/emvee:/bin/bash
nls:x:1016:1016::/home/nls:/bin/bash
noname:x:1017:1017::/home/noname:/bin/bash
nolose:x:1018:1018::/home/nolose:/bin/bash
sancelisso:x:1019:1019::/home/sancelisso:/bin/bash
ruycr4ft:x:1020:1020::/home/ruycr4ft:/bin/bash
tasiyanci:x:1021:1021::/home/tasiyanci:/bin/bash
lanz:x:1022:1022::/home/lanz:/bin/bash
pylon:x:1023:1023::/home/pylon:/bin/bash
wwfymn:x:1024:1024::/home/wwfymn:/bin/bash
whitecr0wz:x:1025:1025::/home/whitecr0wz:/bin/bash
bit:x:1026:1026::/home/bit:/bin/bash
infayerts:x:1027:1027::/home/infayerts:/bin/bash
rijaba1:x:1028:1028::/home/rijaba1:/bin/bash
cromiphi:x:1029:1029::/home/cromiphi:/bin/bash
gatogamer:x:1030:1030::/home/gatogamer:/bin/bash
ch4rm:x:1031:1031::/home/ch4rm:/bin/bash
aceomn:x:1032:1032::/home/aceomn:/bin/bash
kerszi:x:1033:1033::/home/kerszi:/bin/bash
d3b0o:x:1034:1034::/home/d3b0o:/bin/bash
avijneyam:x:1035:1035::/home/avijneyam:/bin/bash
zayotic:x:1036:1036::/home/zayotic:/bin/bash
kaian:x:1037:1037::/home/kaian:/bin/bash
c4rta:x:1038:1038::/home/c4rta:/bin/bash
boyras200:x:1039:1039::/home/boyras200:/bin/bash
waidroc:x:1040:1040::/home/waidroc:/bin/bash
ziyos:x:1041:1041::/home/ziyos:/bin/bash
b4el7d:x:1042:1042::/home/b4el7d:/bin/bash
rpj7:x:1043:1043::/home/rpj7:/bin/bash
h1dr0:x:1044:1044::/home/h1dr0:/bin/bash
catch_me75:x:1045:1045::/home/catch_me75:/bin/bash
josemlwdf:x:1046:1046::/home/josemlwdf:/bin/bash
skinny:x:1047:1047::/home/skinny:/bin/bash
0xeex75:x:1048:1048::/home/0xeex75:/bin/bash
0xh3rshel:x:1049:1049::/home/0xh3rshel:/bin/bash
0xjin:x:1050:1050::/home/0xjin:/bin/bash
```
这个机器上的用户相当多

搜一下密码可以搜出来两个
```sh
patrick@slackware:/home$ find ./ -name *pass* -type f 2>/dev/null
./claor/mypass.txt
./kretinga/mypass.txt
patrick@slackware:/home$ cat ./claor/mypass.txt
JRksNe5rWgis
patrick@slackware:/home$ cat ./kretinga/mypass.txt
lpV8UG0GxKuw
```

之后做的就是不断切换用户再去搜索密码
```sh
rpj7/wP26CtkDby6J
```

![Pasted image 20250217202131.png](/img/user/picture/Pasted%20image%2020250217202131.png)
```sh
rpj7@slackware:~$ ls
mypass.txt  user.txt
rpj7@slackware:~$ cat user.txt
HMV{Th1s1s1Us3rFlag}






rpj7@slackware:~$ id
uid=1043(rpj7) gid=1043(rpj7) groups=1043(rpj7),1044(h1dr0)

```

# 提权2

注意到上面user.txt文件下面有一大段空行
并且文件大小也不太正常
```sh
rpj7@slackware:~$ cat -A user.txt
HMV{Th1s1s1Us3rFlag}^I     ^I      ^I ^I   ^I     ^I       ^I       $
    ^I      ^I^I    ^I       ^I^I^I ^I   ^I       $
       ^I ^I     ^I       ^I      ^I    ^I      ^I     ^I    $
^I     ^I    ^I ^I    ^I     ^I      ^I       ^I   ^I  $
     ^I   ^I   ^I     ^I     ^I   ^I ^I^I      ^I $
     ^I       ^I       ^I    ^I      ^I    ^I     ^I  ^I $
^I       ^I     ^I $

```
所以这文件中可能存在隐写
先把它下到本地看看
```sh
scp -P 1 rpj7@192.168.4.7:/home/rpj7/user.txt ./user.txt
```

```sh
┌──(root㉿kali)-[~/workspace/pentest/Slackware]
└─# stegsnow -C user.txt
To_Jest_Bardzo_Trudne_Haslo 
```

这个隐藏数据就是root密码
```sh
rpj7@slackware:~$ su
Password:
root@slackware:/home/rpj7# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),17(audio)
```


# whereflag

```sh
root@slackware:~# ls -al
total 5
drwx--x---  6 root root 232 Mar 11  2024 .
drwxr-xr-x 23 root root 536 Mar 10  2024 ..
lrwxrwxrwx  1 root root   9 Mar 10  2024 .bash_history -> /dev/null
drwx------  3 root root  72 Mar 11  2024 .cache
drwx------  3 root root  72 Feb 16  2024 .config
drwx------  2 root root 136 Feb 16  2024 .gnupg
lrwxrwxrwx  1 root root   9 Mar 11  2024 .lesshst -> /dev/null
drwx------  3 root root  72 Feb 16  2024 .local
-r--------  1 root root  72 Mar 11  2024 roo00oot.txt
root@slackware:~# cat roo00oot.txt
There is no root flag here, but it is somewhere in the /home directory.
```

```sh
root@slackware:/home# find . -type f -exec grep -l "HMV" {} \;
./rpj7/user.txt
./0xh3rshel/.screenrc
root@slackware:/home# grep "HMV" ./0xh3rshel/.screenrc
# Here is a flag for root: HMV{SlackwareStillAlive}

```






# 参考资料


+ http://162.14.82.114/index.php/627/04/25/2024/



