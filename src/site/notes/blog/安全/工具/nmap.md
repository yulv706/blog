---
{"dg-publish":true,"dg-path":"安全/工具/nmap.md","permalink":"///nmap/","title":"nmap"}
---

# 主机发现

## 取消端口扫描

`-sn`


## 基于ICMP的主机发现

主要包括三类

+ `-PE`
	+ ping请求(ICMP Type 8/Echo)和ping返回(ICMP Type 0)
+ `-PP`
	+ 时间戳(timestamp)请求(ICMP Type 13)和时间戳返回 (ICMP Type 14)
+ `-PM`
	+ 地址掩码(address mask)请求(ICMP Type 17)和地址掩码返回


许多防火墙会阻止 ICMP 回显;新版本的 MS Windows 配置了主机防火墙，默认情况下会阻止 ICMP 回显请求。
请记住，如果目标位于同一子网上，则 ARP 查询将位于 ICMP 请求之前。


## 基于ARP的主机发现

**`-PR`**


## 基于**UDP、TCP**的主机发现

### **TCP SYN Ping**

**`-PS`**
这个模式将默认向80端口发送一个SYN标志的数据包
打开的端口应使用 SYN/ACK （Acknowledge） 进行回复;关闭的端口将导致 RST （Reset）。
但是在主机发现阶段，**我们只检查是否会收到任何响应来推断主机是否已启动。** 端口的具体状态在这里并不重要。

其中TCP三次握手原理如下：
![Pasted image 20240911143008.png](/img/user/picture/Pasted%20image%2020240911143008.png)
注意：
特权用户（root 和 sudoers）可以发送 TCP SYN 数据包，即使端口打开，也不需要完成 TCP 三次握手，但是非特权用户别无选择，只能完成 3 次握手。

### **TCP ACK Ping**
**`-PA`**
这个模式将默认向80端口发送一个ACK标志的数据包
目标使用设置的 RST 标志进行响应，因为带有 ACK 标志的 TCP 数据包不属于任何正在进行的连接。预期响应用于检测目标主机是否已启动。

### **UDP Ping**
**`-PU`**
与 TCP SYN ping 相反，将 UDP 数据包发送到开放端口预计不会导致任何回复。但是，**如果我们将 UDP 数据包发送到关闭的 UDP 端口，我们预期会收到 ICMP 端口无法访问的数据包;这表示目标系统已启动且可用。**
![Pasted image 20240911144826.png](/img/user/picture/Pasted%20image%2020240911144826.png)![Pasted image 20240911144830.png](/img/user/picture/Pasted%20image%2020240911144830.png)



## 反向DNS查询

反向 DNS 查找（Reverse DNS Lookup）是一种通过 IP 地址查找对应域名的过程。
Nmap 的默认会进行反向DNS查询。因为主机名可以显示很多信息，所以这可能是一个有用的步骤。
但是，如果您不想发送此类 DNS 查询，请使用 **-n** 跳过此步骤。


默认情况下只会对在线主机进行反向DNS查询，如果想要对所有主机进行反向DNS，可以使用`-R`
如果要使用特定的 DNS 服务器，可以添加 `--dns-servers`

# 基本端口扫描

nmap对于端口状态有六种状态：
+ **open**：表示服务正在监听指定的端口。
+ **Closed**：表示没有服务正在侦听指定的端口，且未被防火墙或其他安全设备/程序阻止
+ **Filtered**：被过滤，表示 Nmap 无法确定端口是打开还是关闭，因为端口无法访问。这种状态通常是由于防火墙阻止 Nmap 到达该端口。
+ **Unfiltered**：表示 Nmap 无法确定端口是打开还是关闭，尽管端口是可访问的。使用 ACK 扫描 -sA 时遇到此状态。
+ **Open|Filtered**：这意味着 Nmap 无法确定端口是打开的还是过滤的。
+ **Closed|Filtered**：这意味着Nmap无法决定一个端口是关闭的还是过滤的。


## TCP标志位

![Pasted image 20240911151932.png](/img/user/picture/Pasted%20image%2020240911151932.png)
从左到右，TCP 标头标志为：
1. **URG**: 紧急标志表示提交的紧急指针很重要。紧急指针指示传入数据是紧急的，并且会立即处理设置了 URG 标志的 TCP 段，而无需考虑必须等待以前发送的 TCP 段。
2. **ACK**:确认标志表示确认编号很重要。它用于确认收到 TCP 段。
3. **PSH**:推送标志要求 TCP 立即将数据传递给应用程序。
4. **RST**: 重置标志用于重置连接。其他设备（如防火墙）可能会发送它来破坏 TCP 连接。当数据发送到主机并且接收端没有服务应答时，也会使用此标志。
5. **SYN**: 标志用于启动 TCP 三次握手并将序列号与其他主机同步。在 TCP 连接建立期间，应随机设置序列号。
6. **FIN**:发送方没有更多数据要发送。


## TCP连接端口扫描
`-sT`
我们感兴趣的是了解 TCP 端口是否打开，而不是建立 TCP 连接。因此，**一旦通过发送 RST/ACK 确认其状态，连接就会被撕裂**.

请注意，我们可以使用 **-F** 来启用快速模式，并将扫描的端口数量从 1000 个最常见的端口减少到 100 个。
值得一提的是，还可以添加 -r 选项，以连续顺序而不是随机顺序扫描端口。在测试端口是否以一致的方式打开时（例如，当目标启动时），此选项非常有用。

## TCP SYN端口扫描
**`-sS`**

注意，它需要特权 （root 或 sudoer） 用户才能运行它

SYN 扫描不需要完成 TCP 三次握手;相反，它会在收到来自服务器的响应后断开连接。由于我们没有建立 TCP 连接，因此这会降低记录扫描的几率。

上半部分是TCP 连接扫描 -sT 流量，可以看到nmap会完成TCP的三次握手
在下图的下半部分，我们看到 SYN 扫描 -sS 如何不需要完成 TCP 三次握手;相反，Nmap在收到SYN/ACK报文后发送RST报文。
![Pasted image 20240911170022.png](/img/user/picture/Pasted%20image%2020240911170022.png)


TCP SYN 扫描是以特权用户身份运行 Nmap、以 root 身份运行或使用 sudo 时的**默认扫描模式**，是一个非常可靠的选择。


## UDP端口扫描

**`-sU`**
如果我们将 UDP 数据包发送到开放的 UDP 端口，则不能期望返回任何回复。因此，将 UDP 数据包发送到开放端口不会告诉我们任何事情。
换句话说，不产生任何响应的 UDP 端口是 Nmap 声明为打开的端口。

![Pasted image 20240911173642.png](/img/user/picture/Pasted%20image%2020240911173642.png)
![Pasted image 20240911173709.png](/img/user/picture/Pasted%20image%2020240911173709.png)


## 调整范围和速度

+ 端口列表：`-p22,80,443`
+ 端口范围：`-p1-1023`
+ 所有 65535 个端口:`-p-`
+ 最常见的 100 个端口:-`F`
+ 最常见10个端口：`--top-ports 10`


速度调整：
+ `-T<0-5>`. `-T0` 是最慢的,  `-T5` 是最快的

可以选择使用 `--min-rate <number>` 和 `--max-rate <number>` 来控制数据包速率

可以使用 `--min-parallelism <numprobes>` 和 `--max-parallelism <numprobes>` 控制探测并行化



# 进阶端口扫描

## 空扫描
**`-sN`**
空扫描不会设置任何标志;所有 6 个 Flag 位都设置为零。
未设置标志的 TCP 数据包在到达开放端口时不会触发任何响应

**空扫描中没有回复表明端口是开放的，或者是防火墙阻止了数据包。**
![Pasted image 20240914102553.png](/img/user/picture/Pasted%20image%2020240914102553.png)
**如果端口关闭，目标服务器使用 RST 数据包进行响应。**
![Pasted image 20240914102628.png](/img/user/picture/Pasted%20image%2020240914102628.png)

## FIN扫描
**`-sF`**
FIN 扫描发送设置了 FIN 标志的 TCP 数据包。
**如果 TCP 端口打开，则不会发送响应。同样，Nmap 无法确定端口是否打开，或者防火墙是否阻止了与此 TCP 端口相关的流量。**
![Pasted image 20240914104033.png](/img/user/picture/Pasted%20image%2020240914104033.png)
**但是，如果端口已关闭，则目标系统应使用 RST 进行响应。**
![Pasted image 20240914104057.png](/img/user/picture/Pasted%20image%2020240914104057.png)


## Xmas扫描
**`-sX`**
Xmas 扫描同时设置 FIN、PSH 和 URG 标志
与 Null scan 和 FIN scan 一样，如果收到 RST 数据包，则表示端口已关闭。否则，它将报告为 open|filtered。
![Pasted image 20240914104343.png](/img/user/picture/Pasted%20image%2020240914104343.png)
![Pasted image 20240914104351.png](/img/user/picture/Pasted%20image%2020240914104351.png)
## ACK扫描
**`-sA`**
ACK 扫描将发送设置了 ACK 标志的 TCP 数据包。**无论端口的状态如何，目标都会使用 RST 响应 ACK。**
**正如预期的那样，我们无法了解哪些端口已打开。**
![Pasted image 20240914142027.png](/img/user/picture/Pasted%20image%2020240914142027.png)
![Pasted image 20240914142421.png](/img/user/picture/Pasted%20image%2020240914142421.png)
如果目标前面有防火墙，这种扫描会很有帮助。因此，根据哪些 ACK 数据包导致了响应，您将了解哪些端口未被防火墙阻止。**换句话说，这种类型的扫描更适合发现防火墙规则集和配置。**

## 窗口扫描
**`-sW`**
TCP 窗口扫描与 ACK 扫描几乎相同;但是，它会检查返回的 RST 数据包的 TCP 窗口字段。在特定系统上，这可能表明端口已打开。
![Pasted image 20240914142501.png](/img/user/picture/Pasted%20image%2020240914142501.png)

## 自定义扫描

如果要尝试使用内置 TCP 扫描类型之外的新 TCP 标志组合，可以使用` --scanflags` 来实现。
例如，如果你想同时设置 SYN、RST 和 FIN，你可以使用 `--scanflags RSTSYNFIN` 来实现。


# 端口后扫描

## 服务检测

**`-sV`**

在 Nmap 命令中添加 `-sV` 将收集并确定开放端口的服务和版本信息。
您可以使用` --version-intensity LEVEL` 控制强度，其中级别范围介于 0（最轻）和 9（最完整）之间。`-sV --version-light` 的强度为 2，而 `-sV --version-all` 的强度为 9。

注意，使用 -sV 将强制 Nmap 进行 TCP 三次握手并建立连接。
换句话说，当选择 -sV 选项时，隐形 SYN 扫描 -sS 是不可能的。

## OS检测

**`-O`**
OS 检测非常方便，但许多因素可能会影响其准确性。首先，Nmap 需要在目标上找到至少一个开放端口和一个封闭端口，以便做出可靠的猜测。此外，由于虚拟化和类似技术的使用越来越多，客户操作系统指纹可能会失真。因此，请始终对操作系统版本持保留态度。

也就是说这种检测并不是很准确


## 路由追踪

**`--traceroute`**

值得一提的是，许多路由器被配置为不发送ICMP超时（Time-to-Live exceeded）消息，这会阻止我们发现它们的IP地址。



# nmap脚本

Nmap 为使用 Lua 语言的脚本提供支持。作为 Nmap 的一部分，Nmap 脚本引擎 （NSE） 是一个 Lua 解释器，允许 Nmap 执行用 Lua 语言编写的 Nmap 脚本。

脚本文件夹一般在下面目录：
`/usr/share/nmap/scripts`


可以指定使用这些已安装脚本中的任何一个或一组;此外，您可以安装其他用户的脚本并将其用于您的扫描。

可以选择使用 `--script=default` 或直接添加` -sC` 来运行默认类别中的脚本。
除了 default 之外，类别还包括 auth、broadcast、brute、default、discovery、dos、exploit、external、fuzzer、intrucious、malware、safe、version 和 vuln。

| 类别       | 描述                                                                 |
|------------|--------------------------------------------------------------------|
| auth       | 进行身份验证相关的脚本，主要用于验证凭据和访问控制。                     |
| broadcast  | 执行广播相关的脚本，向网络中的多个主机发送请求并收集响应信息。               |
| brute      | 进行暴力破解攻击的脚本，用于尝试多种凭据组合以获得访问权限。                 |
| default    | Nmap默认使用的脚本，执行常见的服务检测、安全审计和漏洞检查。                 |
| discovery  | 用于主机发现、服务发现和网络信息收集的脚本。                             |
| dos        | 执行拒绝服务攻击的脚本，旨在通过消耗资源使目标无法正常服务。                 |
| exploit    | 执行漏洞利用的脚本，用于触发目标系统中的已知漏洞进行渗透。                    |
| external   | 利用外部资源进行检测的脚本，如查询在线数据库或第三方服务。                    |
| fuzzer     | 用于模糊测试的脚本，通过向目标服务发送畸形数据来检测潜在的漏洞。                |
| intrusive  | 可能对目标系统产生负面影响的脚本，这些脚本有时会引起系统异常或不稳定。            |
| malware    | 检测目标是否感染恶意软件或病毒的脚本，检查已知的恶意软件特征。                   |
| safe       | 被认为对目标不会产生负面影响的脚本，适用于安全测试和常规扫描。                   |
| version    | 用于检测服务版本信息的脚本，帮助确定运行的应用程序的版本号。                    |
| vuln       | 用于检测目标系统是否存在已知漏洞的脚本，通常与安全审计和补丁管理相关。             |
某些脚本属于多个类别。此外，一些脚本对服务发起暴力攻击，而另一些脚本则发起 DoS 攻击和漏洞利用系统。**因此，如果您不想使服务崩溃或利用它们，那么在选择要运行的脚本时必须小心。**

您还可以使用 `--script "SCRIPT-NAME"` 或模式 （如 `--script "ftp*"`） 指定脚本，其中包括 ftp-brute。如果您不确定脚本的作用，可以使用文本阅读器 （如 less） 或文本编辑器打开脚本文件。对于 ftp-brute，它指出：“对 FTP 服务器执行暴力密码审核。您必须小心，因为某些脚本非常具有侵入性。此外，某些脚本可能适用于特定服务器，如果随机选择，会浪费您的时间而没有任何好处。


# 结果保存

有四种结果保存形式：
1. Normal
2. Grepable (`grep`able)
3. XML
4. Script Kiddie(不推荐)



## Normal

顾名思义，正常格式类似于扫描目标时在屏幕上获得的输出。您可以使用` -oN FILENAME` 以普通格式保存扫描内容;N 代表正常。


## Grepable
“grepable”格式因命令`grep`而得名；`grep`代表“Global Regular Expression Printer”（全局正则表达式打印机）。简单来说，这种格式可以高效地过滤扫描输出中的特定关键词或术语。你可以使用`-oG FILENAME`将扫描结果保存为grepable格式。上述以普通格式显示的扫描输出在控制台中以grepable格式显示。普通输出有21行，而grepable输出只有4行。主要原因是Nmap希望在用户使用`grep`时使每一行都具有意义和完整性。
**因此，在grepable输出中，行的长度很长，相比普通输出，不便于阅读。**

`grep`的一个使用示例是 `grep KEYWORD TEXT_FILE`；这个命令将显示所有包含指定关键词的行。让我们比较在普通输出和grepable输出上使用`grep`的结果。你会注意到，普通输出并没有提供主机的IP地址，而是返回了`80/tcp open http nginx 1.6.2`，这在浏览多个系统的扫描结果时非常不方便。而grepable输出在每一行中都提供了足够的信息，如主机的IP地址，使其更加完整。

## XML

第三种格式是XML格式。你可以使用 `-oX FILENAME` 将扫描结果保存为XML格式。XML格式最方便用于在其他程序中处理输出。更方便的是，你可以使用 `-oA FILENAME` 同时将扫描结果保存为所有三种格式，等同于结合 `-oN`（普通格式）、`-oG`（grepable格式）和 `-oX`（XML格式）。




