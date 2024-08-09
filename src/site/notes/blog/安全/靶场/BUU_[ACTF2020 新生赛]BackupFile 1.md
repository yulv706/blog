---
{"dg-publish":true,"title":"BUU_[ACTF2020 新生赛]BackupFile 1","dg-path":"安全/靶场/BUU_[ACTF2020 新生赛]BackupFile 1.md","permalink":"/安全/靶场/BUU_[ACTF2020 新生赛]BackupFile 1/","dgPassFrontmatter":true}
---

![Pasted image 20240809161546.png](/img/user/picture/Pasted%20image%2020240809161546.png)
扫描目录后发现代码备份`index.php.bak`
下载后查看
```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}

```
也就是通过url接收变量key值

要使 PHP 代码中的 `$key` 等于字符串 `$str`，首先要理解 PHP 中的类型转换。PHP 是一种弱类型语言，它会在需要时进行类型转换。在比较字符串和整数时，PHP 会尝试将字符串转换为数字，然后再进行比较。

对于字符串 `$str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";`，PHP 会从字符串的开头解析尽可能多的数字部分，直到遇到第一个非数字字符为止。因此，在这种情况下，PHP 会将 `$str` 转换为 `123`（字符串开头的数字部分），然后再与 `$key` 进行比较。

因此，为了使 `$key == $str` 成立，`$key` 的值应为 `123`。

得到flag
![Pasted image 20240809161742.png](/img/user/picture/Pasted%20image%2020240809161742.png)