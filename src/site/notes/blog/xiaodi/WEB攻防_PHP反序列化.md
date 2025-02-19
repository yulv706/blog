---
{"dg-publish":true,"title":"WEB攻防_PHP反序列化","dg-path":"xiaodi/WEB攻防_PHP反序列化.md","permalink":"/xiaodi/WEB攻防_PHP反序列化/","dgPassFrontmatter":true}
---

# 什么是序列化
**在编程中，序列化是将对象转换为字节流或其他格式，以便存储或传输的过程，**
**而反序列化则是将这些格式化的数据重新转换回内存中的对象。**

`serialize()`将对象转换成一个字符串
`unserialize()`将字符串还原成一个对象
![Pasted image 20240929150336.png](/img/user/picture/Pasted%20image%2020240929150336.png)
(这里这个对象长度指的是这个对象名称的长度，也就是info是4个字符)


# 常见PHP魔术方法

 `__construct()`
**当对象被创建(new)的时候会自动调用**

`__destruct()`
**当对象被销毁时会被自动调用**
+ 一种是主动销毁(unset)
+ 一种是由系统自动销毁

`__sleep()`
**serialize()执行时被自动调用**

`__wakeup()`
**unserialize()时会被自动调用**

`__invoke()`
**当尝试以调用函数的方法调用一个对象时会被自动调用**

`__toString()`
**把类当作字符串使用时触发**

`__call()`
**调用某个方法,若方法存在,则调用;若不存在,则会去调用__call函数。**

`__callStatic()`
**在静态上下文中调用不可访问的方法时触发**
 
`__get()`
**读取对象属性时,若存在,则返回属性值;若不存在，则会调用__get函数**

`__set()` 
**设置对象的属性时,若属性存在,则赋值;若不存在,则调用__set函数。**

`__isset()`
**在不可访问的 属性上调用isset()或empty()触发**
在正常调用isset()的时候会返回0/1来表示这个属性有没有被设置
但是在调用私有属性之类的不可访问属性时，就会调用类中定义好的`__isset()`函数

`__unset()`
**在不可访问的属性上使用unset()时触发**

`__set_state()`
**调用var_export()导出类时，此静态方法会被调用**

`__clone()`
**当对象复制完成时调用**

`__autoload()`
**尝试加载未定义的类**

`__debugInfo()`
**打印所需调试信息**

# 属性类型

有**三种**变量类型：
+ `public`：在本类内部、外部类、子类都可以访问
+ `protect`：只有本类或子类或父类中可以访问
+ `private`：只有本类内部可以使用

这三种数据类型在序列化之后会有一些差异，来帮助php反序列化的时候去识别每个变量的类型
+ `public`属性序列化的时候格式是正常成员名
+ `protect`属性序列化的时候格式是`%00*%00成员名`
+ `private`属性序列化的时候格式是`%00类名%00成员名`


# 字符逃逸

字符逃逸我认为主要是依靠`str_replace()`函数，也就是它会将一个a长度的字符串修改为b长度的字符串(`a!=b`)，然后将我们要修改的信息传入并将序列化字符串闭合
我们知道在反序列化时需要每一个变量和他前面长度相符才能反序列化成功
我们下面讨论两种情况来分析他是如何进行字符逃逸的：
## 整体概述
我们用下面代码演示(我们可以修改`uname`，目的是将`password`修改为`yes`)：
```php
<?php
class A
{
    public $uname = '12';
    public $password = 'no';
}

function filter($string){
    return str_replace('12', '123', $string);
}

$ser = filter(serialize(new A()));
$test = unserialize($ser);
echo $test->password;
```

也就是说我们大致需要将序列化字符串共造成这样：
```
O:1:"A":2:{s:5:"uname";s:2:"12";s:8:"password";s:3:"yes";}
```

我们要知道一个序列化字符串的结束是以`}`符号为标志的，所以我们就有一个思路，那就是能不能在某一个值中将序列化字符串闭合来达到我们的目的

正常情况下，如果我们将uname修改成下面：
`12";s:8:"password";s:3:"yes";}`
构造的序列化字符串为：
```
O:1:"A":2:{s:5:"uname";s:30:"12";s:8:"password";s:3:"yes";}";s:8:"password";s:2:"no";}
```
在反序列化时会将`O:1:"A":2:{s:5:"uname";s:30:"12";s:8:"password";s:3:"yes";}`看作一个序列化字符，我们发现这里确实将我们不想要的数据闭合在了外面，把我们想要的password闭合在了里面
但是有一个大问题，那就是我们发现uname的长度不对，这将导致反序列化无法成功
但是如果存在将序列化字符串中的某个字符串替换为另一个不同长度的，就给我们机会去将这个反序列化字符串合理化


## 字符增多
也就是它会将一个短字符替换为一个长字符
```php
<?php
class A
{
    public $uname = '12';
    public $password = 'no';
}

function filter($string){
    return str_replace('12', '123', $string);
}

$ser = filter(serialize(new A()));
$test = unserialize($ser);
echo $test->password;
```
我们需要在uname中包含`";s:8:"password";s:3:"yes";}`,也就是多了28个字符，而一个`12`会替换为`123`也就是增多1字符
我们可以构造uname:
```
12121212121212121212121212121212121212121212121212121212";s:8:"password";s:3:"yes";}
```

![Pasted image 20241009110040.png](/img/user/picture/Pasted%20image%2020241009110040.png)
为什么会这样的，我们看下过滤后的序列化字符串
```
O:1:"A":2:{s:5:"uname";s:84:"123123123123123123123123123123123123123123123123123123123123123123123123123123123123";s:8:"password";s:3:"yes";}";s:8:"password";s:2:"no";}
```

反序列化时会将
```
O:1:"A":2:{s:5:"uname";s:84:"123123123123123123123123123123123123123123123123123123123123123123123123123123123123";s:8:"password";s:3:"yes";}
```
看作一个序列化字符串，发现由于字符串替换，uname的长度变得正确

### 练习靶场

[[blog/安全/靶场/CTFshow_月饼杯_web1_此夜圆\|CTFshow_月饼杯_web1_此夜圆]]

## 字符减少

字符减少的逃逸思路是依靠两个相邻的变量，在后面的变量中我们完整包含我们想要包含的信息，并且还要尝试和前面的变量进行闭合，还要将不要的信息用`}`隔断。而前面的变量就要包含会缩短的字符来进行长度调整
还要注意，这样的话变量数量应该会减少，所以我们还在在后面的值中添加一个占位量，可以是下面内容`s:1:"1";s:1:"2";`

### 练习靶场
[[blog/安全/靶场/BUU_[安洵杯 2019]easy_serialize_php 1\|BUU_[安洵杯 2019]easy_serialize_php 1]]



# 原生类

**基本思路也就是利用PHP环境中内置的类的魔术方法**
拿一个简单的例子：
```php
<?php
$a = unserialize($_GET['whoami']);
echo $a;//把类当作字符串使用时触发__toString
?>
```
会发现代码中并没有明确定义的类。
首先要知道php环境中是有一些原本就有的类，而我们经常利用的有`Error  Exception SoapClient DirectoryIterator SimpleXMLElement`这几个，而我们如果传入一个原生类的序列化字符串给这个程序，他就会调用原生类的`__toString`魔术方法,如果我们对其加以利用，就能实现攻击

## Error 内置类

可以利用其中的`__toString`进行**XSS**攻击
- 适用于php7版本

简单POC：
```php
<?php
$a = new Error("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>
```



## Exception 内置类

可以利用其中的`__toString`进行**XSS**攻击
- 适用于php5、7版本

简单POC：
```php
<?php
$a = new Exception("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>
```


**简单练习：**
[[blog/安全/靶场/BUU_[BJDCTF 2nd]xss之光\|BUU_[BJDCTF 2nd]xss之光]]

## 使用 Error/Exception 内置类绕过哈希比较

看下面代码：
```php
<?php
$a = new Error("payload",1);$b = new Error("payload",2);//注意需要在同一行定义
echo $a;
echo "\r\n\r\n";
echo $b;
```
输出如下：
```php
Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}

Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}
```

可见，`$a` 和 `$b` 这两个错误对象本身是不同的，但是 `__toString` 方法返回的结果是**相同的**。

知道了前置知识，我们下面看例题：
[[blog/安全/靶场/BUU_[极客大挑战 2020]Greatphp\|BUU_[极客大挑战 2020]Greatphp]]




# 框架类反序列化

也就是这种不是原生代码导致的反序列化漏洞，而是一些开发框架，比如thinkphp，Yii等框架导致的反序列化漏洞


例题：
[[blog/安全/靶场/BUU_[安洵杯 2019]iamthinking\|BUU_[安洵杯 2019]iamthinking]]





# 靶场练习

[[blog/安全/靶场/CTFshow_反序列化1\|CTFshow_反序列化1]]
[[CTFshow_反序列化2(懒，没做完)\|CTFshow_反序列化2(懒，没做完)]]
[[CTFshow_反序列化3(累，没做完)\|CTFshow_反序列化3(累，没做完)]]
[[CTFshow_反序列化4(困，没做完)\|CTFshow_反序列化4(困，没做完)]]
[[blog/安全/靶场/BUU_[极客大挑战 2019]PHP1\|BUU_[极客大挑战 2019]PHP1]]
