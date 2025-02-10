---
{"dg-publish":true,"title":"WEB攻防_SQL注入","dg-path":"xiaodi/WEB攻防_SQL注入.md","permalink":"/xiaodi/WEB攻防_SQL注入/","dgPassFrontmatter":true}
---

# 数据库框架

+ maysql
	+ ==**数据库A**==
		+ **表a**
			+ 列名1
				+ 数据
			+ 列名2
				+ 数据
		+ **表b**
			+ 列名1
				+ 数据
			+ 列名2
				+ 数据
	+ ==**数据库B**==
		+ **表c**
			+ 列名1
				+ 数据
			+ 列名2
				+ 数据
		+ **表d**
			+ 列名1
				+ 数据
			+ 列名2
				+ 数据


## 保存关键信息数据库

在mysql数据库中，有一个固定的数据库**information_schema**保存着有关数据库的很多信息

+ **表SCHEMATA**(5列)
	+ 保存着所有数据库名称
	+ 关键列：SCHEMA_NAME
![Pasted image 20240726111753.png](/img/user/picture/Pasted%20image%2020240726111753.png)

+ **表TABLES**
	+ 记录**表名信息**的表
	+ 关键列：TABLE_SCHEMA、TABLE_NAME
![Pasted image 20240726112739.png](/img/user/picture/Pasted%20image%2020240726112739.png)


+ **表columns**
	+ 记录**列名**信息表
	+ 关键列：TABLE_SCHEMA、TABLE_NAME、COLUMN_NAME
![Pasted image 20240726113023.png](/img/user/picture/Pasted%20image%2020240726113023.png)

# SQL注入类型

+ 基于正常回显
	联合查询 union select
	
+ 盲注
	+ 布尔型盲注
    + 基于时间盲注sleep()
    + 基于错误回显
	    floor()
	    extractvalue()
	　　updatexml()





# masql 常规注入查询

获取相关数据：

1、数据库版本-看是否符合information_schema查询-version()
低版本没有information_schema数据库

2、数据库用户-看是否符合ROOT型注入攻击-user()

3、当前操作系统-看是否支持大小写或文件路径选择-@@version_compile_os

4、数据库名字-为后期猜解指定数据库下的表，列做准备-database()


[[blog/安全/靶场/墨者_XFF注入\|墨者_XFF注入]]
[[blog/安全/靶场/CTFshow_SQL注入_无过滤注入\|CTFshow_SQL注入_无过滤注入]]
# SQL盲注

## 布尔盲注报错

基于布尔型SQL盲注即在SQL注入过程中，应用程序仅仅返回`True`（页面）和`False`（页面）。
这时，我们无法根据应用程序的返回页面得到我们需要的数据库信息。但是可以通过构造逻辑判断（比较大小）来得到我们需要的信息。

>[!note]
>*也就是我们注入查询的结果没有办法直接显示，所以我们可以依靠来构造查语句的**正确性**来**猜测爆破**信息*

常用语句：

```sql
and length(database())=7;

and left(database(),1)='p';

and left(database(),2)='pi';

and substr(database(),1,1)='p';

and substr(database(),2,1)='i';

and ord(left(database(),1))=112;
```




### practice-DVWA

[[blog/安全/靶场/DVWA-SQL盲注-easy\|DVWA-SQL盲注-easy]]


## 报错盲注

**使SQL语句报错的语法，用于注入结果无回显但错误信息有输出的情况。**

### 通过UpdateXml报错

```sql
and updatexml(1,concat(0x7e,(SELECT version()),0x7e),1)
```


### 通过ExtractValue报错

```sql
and extractvalue(1, concat(0x7e,(SELECT version()),0x7e))
```


### 通过floor报错

```sql
//没看懂
```

[[blog/安全/靶场/自建环境_SQL报错盲注测试\|自建环境_SQL报错盲注测试]]


https://juejin.cn/post/7156744293988696095
https://www.jianshu.com/p/bc35f8dd4f7c

## 延迟盲注

```sql
and if(1=1,sleep(5),0)

or if(1=1,sleep(5),0)

or if(ord(left(database(),1))=107,sleep(2),0)
```




# 二次注入

二次注入可以理解为，**攻击者构造的恶意数据存储在数据库**后，**恶意数据被读取并进入到SQL查询语句**所导致的注入。防御者即使对用户输入的恶意数据进行**转义**，当数据插入到数据库中时被处理的数据又被还原，Web程序调用存储在数据库中的恶意数据并执行SQL查询时，就发生了SQL二次注入。

也就是说一次攻击造成不了什么，但是两次配合起来就会早成注入漏洞。



![Pasted image 20240802112705.png](/img/user/picture/Pasted%20image%2020240802112705.png)



[[随记/自建环境_二次注入测试_{2024-08-02}\|自建环境_二次注入测试_{2024-08-02}]]



# 堆叠注入

堆叠注入触发的条件很苛刻，因为堆叠注入原理就是通过结束符同时执行多条sql语句，

例如php中的mysqli_multi_query函数。与之相对应的mysqli_query()只能执行一条SQL，所以要想目标存在堆叠注入,在目标主机存在类似于mysqli_multi_query()这样的函数,根据数据库类型决定是否支持多条语句执行.


[[blog/安全/靶场/BUU_强网杯随便注\|BUU_强网杯随便注]]


# 带外注入

说是几乎见不到，先暂时不搞了
	


# sqlmap

[[blog/安全/工具/sqlmap\|sqlmap]]

