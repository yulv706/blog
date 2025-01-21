---
{"dg-publish":true,"dg-path":"THM/netcat shell稳定.md","permalink":"/THM/netcat shell稳定/","title":"netcat shell稳定"}
---

# 前置知识

**交互性**

交互式 shell，这些允许您在执行程序后与程序进行交互。例如，采用SSH登录提示：
![Pasted image 20240919000144.png](/img/user/picture/Pasted%20image%2020240919000144.png)
在这里您可以看到它_以交互方式_询问用户输入“是”或“否”才能继续连接。这是一个交互式程序，需要交互式 shell 才能运行。

在非交互式 shell 中，您只能使用不需要用户交互才能正常运行的程序。不幸的是，大多数简单的反向和绑定 shell 都是非交互式的，这可能会使进一步的利用变得更加棘手。让我们看看当我们尝试在非交互式 shell 中运行SSH时会发生什么：
![Pasted image 20240919000220.png](/img/user/picture/Pasted%20image%2020240919000220.png)
请注意， `whoami`命令（非交互式）执行得很好，但`ssh`命令（_交互式_）根本没有给我们任何输出。


# 技术1：python

### 步骤一：

首先，使用以下命令通过 Python 生成一个功能更强大的 bash shell：

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

> **注意**：某些目标可能需要指定 Python 的版本。如果是这种情况，将 `python` 替换为 `python2` 或 `python3`，具体取决于需要的版本。

此时，shell 的界面会稍微美观一些，但我们仍然无法使用 Tab 键自动补全或方向键，按 `Ctrl + C` 仍然会杀掉这个 shell。

---

### 步骤二：

接着，输入以下命令：

```bash
export TERM=xterm
```

这将允许我们使用终端命令，例如 `clear`。

---

### 步骤三（最重要的一步）：

1. 使用 `Ctrl + Z` 将 shell 置于后台。
2. 回到自己的终端，执行以下命令：

```bash
stty raw -echo; fg
```

这会完成两件事：

- 关闭自己终端的回显，启用 Tab 自动补全、方向键操作，并让 `Ctrl + C` 可以终止进程。
- 将 shell 前台化，从而完成整个过程。

![Pasted image 20240918235909.png](/img/user/picture/Pasted%20image%2020240918235909.png)

# 技术2：rlwrap
rlwrap 是一个程序，简单来说，它让我们在收到 shell 后立即访问历史记录、制表符自动完成和箭头键_；_但是，如果您希望能够在 shell 内使用 Ctrl + C，则仍然必须使用_一些_手动稳定功能。 Kali 上默认没有安装 rlwrap，因此首先使用`sudo apt install rlwrap`安装它。

```shell
rlwrap nc -lvnp <port>
```
在我们的 netcat 监听器前面加上“rlwrap”给我们一个功能更齐全的 shell。这种技术在处理 Windows shell 时特别有用，否则众所周知，Windows shell 很难稳定。在处理 Linux 目标时，可以通过使用与先前技术的第三步中相同的技巧来完全稳定：使用 Ctrl + Z 将 shell 设置为后台，然后使用`stty raw -echo; fg`稳定并重新进入 shell。

