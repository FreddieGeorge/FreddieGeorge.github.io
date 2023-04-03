---
title: shell脚本笔记
date: 2023-03-02 13:47:08
tags:
- shell
- Linux
categories: 
- Linux
excerpt: shell脚本常用语法、技巧
---

## 前言

最近在工作中编写了几个脚本方便自己的工作流程，在其中也遇到了几个问题，也积累了一些方法和技巧。这篇博客中记录一些shell脚本相关的知识点，但不会包含最基本的内容。

## 循环

### for循环

[bash-for-loop](https://www.cyberciti.biz/faq/bash-for-loop/)

for循环的基本语法如下

```shell
for varname in list
do
    # do something
done
```

for循环的主要用途是将list的元素取出，依序放到varname中，然后执行do和done之间的命令，直到所有元素被取出。

for常用来遍历文件，或者遍历整数等

```shell
# file loop
for VARIABLE in file1 file2 file3
do
    command1 on $VARIABLE
    command2
    commandN
done

# var loop
for VARIABLE in 1 2 3 4 5 .. N
do
    command1
    command2
    commandN
done

# 这种是更高级的使用方法，详细见参考资料
for OUTPUT in $(Linux-Or-Unix-Command-Here)
do
    command1 on $OUTPUT
    command2 on $OUTPUT
    commandN
done
```

### while循环

[玩转Bash脚本：循环结构之while循环](https://blog.csdn.net/guodongxiaren/article/details/43341769)

while循环的基本语法如下

```shell
while condition
do
    # do something
done
```

while循环经常用到的是搭配转向输入，例如

```shell
#!/bin/bash
while grep "123"
do
  # do somethin
done < /path/to/file
```
这个脚本是用来将文件内容逐行传递给while后面的指令，然后再执行循环体。

当循环体为空,则可以看成`cat /path/to/file | grep "123"`

搭配转向的使用更多是结合`read`，用来将文件内容逐行去除，赋给read后面的变量。

此外还有一个until循环，就是while循环的相反用法，while测试的是真值，until判断的是假值

## 分支

shell的分支包括两种，`if else` 和 `case in`。

### if esle

基本语法如下
```shell
if [ command_1 ];then
    # do something
elif [ command_2 ];then
    # do something
else
    # do something
fi
```

我比较喜欢这种写在同一行的写法，看上去比较像c的if和else，这种写法有几个注意事项
* if和`[]`之间要有空格
* `[`的后面和`]`的前面必须要有空格
* if 和 then如果在同一行需要添加 `;`
* `]` 和 `;` 之间不能有空格

接下来是一些比较常用的判断式子，有些我会直接用类似C语言的方式描述他们的作用,参考自[该文章](https://www.jb51.net/article/235932.htm)

#### 常用数字判断

|  表达式   | 作用  | 指令含义 |
|  :--:  | :--:  | :--: |
| [ int1 -eq int2 ]  | int1 == int2 | equal |
| [ int1 -ne int2 ]  | int1 != int2 | not equal |
| [ int1 -gt int2 ]  | int1 >  int2 | grearter than |
| [ int1 -ge int2 ]  | int1 >= int2 | greater equal |
| [ int1 -lt int2 ]  | int1 <  int2 | less than |
| [ int1 -le int2 ]  | int1 <= int2 | less equal |

#### 常用字符串判断

|  表达式   | 作用  |
|  :--:  | :--:  |
| [ -z str ]  | 如果str为空则返回真 |
| [ -n str ]  | 如果str不为空则返回真 |
| [ str1 == str2 ]  | 字符串是否相等 |
| [ str1 != str2 ]  | 字符串是否不相等 |

#### 常用文件判断

文件类型判断

> 该表格内指令都会先判断文件是否存在，如果不存在直接返回假

|  表达式   | 作用  |
|  :--:  | :--:  |
| [ -e FILE ]  | 判断文件或者目录是否存在 |
| [ -b FILE ]  | 判断文件是否为块设备文件 |
| [ -c FILE ]  | 判断文件是否为字符设备文件 |
| [ -d DIR ]  | 判断文件是否为目录文件 |
| [ -f FILE ]  | 判断文件是否为普通文件 |
| [ -L FILE ]  | 判断文件是否为符号链接文件 |
| [ -p FILE ]  | 判断文件是否为管道文件 |
| [ -s FILE ]  | 判断文件是否为空 |
| [ -S FILE ]  | 判断文件是否为套接字文件 |

文件权限判断

> 该表格内指令都会先判断文件是否存在，如果不存在直接返回假

|  表达式   | 作用  |
|  :--:  | :--:  |
| [ -r FILE ]  | 判断文件是否有读权限 |
| [ -w FILE ]  | 判断文件是否有写权限 |
| [ -x FILE ]  | 判断文件是否有执行权限 |

文件之间比较

|  表达式   | 作用  |
|  :--:  | :--:  |
| [ FILE1 -nt FILE2 ]  | 判断文件1修改时间是否比文件2新 |
| [ FILE1 -ot FILE2 ]  | 判断文件1修改时间是否比文件2旧 |
| [ FILE1 -ef FILE2 ]  | 判断文件1是否与文件2的Inode一致，常用于判断硬链接 |

#### 逻辑判断

|  表达式   | 作用  |
|  :--:  | :--:  |
| [ cmd1 -a cmd2 ]  | 逻辑与 |
| [ cmd1 -o cmd2 ]  | 逻辑或 |
| [ ! cmd2 ]  | 逻辑非 |

### case in

case in 在判断条件比较简单，分支较多时比较好用

模板如下

```shell
case expression in
    pattern1)
        # do something
        ;;
    pattern2|pattern3)
        # do something
        ;;
    *)
        # do default
        ;; # this ";;" can be omitted
esac
```

#### 正则表达式

`case in`的pattern部分支持一些简单的正则表达式，具体见表

| 格式 | 说明 |
| :-: | :-: |
| * | 任意字符 |
| [abc] | a,b,c中的任意一个字符 |
| [m-n] | 从m到n的任意一个字符 |
| \| | 相当于逻辑或 |

## 路径处理

<!-- 待更新 -->

一个shell脚本需要获取的最关键的路径主要有：shell脚本所在位置的绝对路径，执行脚本的路径。

### 脚本所在位置的绝对路径

shell脚本的路径可以使用dirname来获取，使用也比较简单

```shell
# dirname
SHELL_FOLDER=$(cd $(dirname "$0");pwd)

# readlink
SHELL_FOLDER=$(dirname $(readlink -f "$0"))
```

### 执行脚本的路径

执行脚本的路径直接使用`pwd`即可


## 打印帮助信息

参考自该[文章](https://developer.aliyun.com/article/972038)，这文章讲得很详细了。

我们只需要在开头部分写入三个`###`起始的注释，利用sed指令即可完成打印，编写一个help函数即可,然后根据后面的参数处理部分使用-h即可打印出来

```shell
### Info
### ......

help() {
    sed -rn 's/^### ?//;T;p;' "$0"
}
```

## 参数
 
参数处理要用到的几个特殊字符

|  字符   | 含义  |
|  :--:  | :--:  |
| $# | 传递到脚本的参数个数  |
| $* | 以一个单字符显示所有向脚本传递的参数 |
| $$ | 脚本程序运行的当给钱进程ID号 |
| $! | 后台运行的最后一个进程的ID号 |
| $@ | 显示所有向脚本传递的参数，但是每个参数都加引号 |
| $0 | 脚本文件名称 |
| $n | 第 n 个参数 |
| $? | 最后指令的退出状态。0表示没有任何错误 |
| $- | shell使用的当前选项 |

参数处理最基本的就是使用 $1 $2 等对执行的参数一个个判断，但是这样的问题就是你必须按照顺序给程序输入参数，而且不能对于可变化的参数无法判断。而getopt和getopts可以解决这个问题，getopt是getopts的拓展，这边只讲getopt的使用

getopt的详细用法

我的参数处理模板为(程序参考自[该博客](https://bummingboy.top/2017/12/19/shell%20-%20%E5%8F%82%E6%95%B0%E8%A7%A3%E6%9E%90%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F(%E6%89%8B%E5%B7%A5,%20getopts,%20getopt)/))

```shell
# 参数处理部分，使用getopt
ARGS=`getopt -o ht --long tar,help -n "$0" -- "$@"`

if [ $? != 0 ]; then
    echo "Terminating..."
    exit 1
fi

# 使用变量方便后续控制流程
OPTION_T=0

#将规范化后的命令行参数分配至位置参数（$1,$2,...)
eval set -- "${ARGS}"

while true
do
    case "$1" in
        -t|--tar) 
            OPTION_T=1
            shift
            ;;
        -h|--help) 
            help
            exit
            shift
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "Internal error!"
            exit 1
            ;;
    esac
done

# 处理完opt后，剩下的参数都在$1 $2...
```

## 颜色

这部分其实是echo相关的，利用echo的`-e` 参数输出有颜色的字符，能输出更多样的脚本打印信息，参考自[该文章](https://www.cnblogs.com/lr-ting/archive/2013/02/28/2936792.html)

```shell
    # 字颜色 30-37
　　echo -e “\033[30m 黑色字 \033[0m” 
　　echo -e “\033[31m 红色字 \033[0m” 
　　echo -e “\033[32m 绿色字 \033[0m” 
　　echo -e “\033[33m 黄色字 \033[0m” 
　　echo -e “\033[34m 蓝色字 \033[0m” 
　　echo -e “\033[35m 紫色字 \033[0m” 
　　echo -e “\033[36m 天蓝字 \033[0m” 
　　echo -e “\033[37m 白色字 \033[0m” 
    # 背景色 40-47 [<back-color>;<font-color>m
　　echo -e “\033[40;37m 黑底白字   \033[0m” 
　　echo -e “\033[41;37m 红底白字   \033[0m” 
　　echo -e “\033[42;37m 绿底白字   \033[0m” 
　　echo -e “\033[43;37m 黄底白字   \033[0m” 
　　echo -e “\033[44;37m 蓝底白字   \033[0m” 
　　echo -e “\033[45;37m 紫底白字   \033[0m” 
　　echo -e “\033[46;37m 天蓝底白字 \033[0m” 
　　echo -e “\033[47;30m 白底黑字   \033[0m” 

```

## 函数


函数其实比较简单，只需要按照如下写即可

```shell
function func_name() {
    # do something
    # return value
}
```

而函数也可以通过`$n`的方式传递参数。

## 字符串

shell中字符串有两种表达方法，分别是单引号和双引号,具体区别不再赘述，我一般直接使用双引号字符串。

### 字符串判断

```shell
${var}              # 变量var的值, 与$var相同
${var-DEFAULT}      # 如果var没有被声明, 那么就以$DEFAULT作为其值 *
${var:-DEFAULT}     # 如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 *
${var=DEFAULT}      # 如果var没有被声明, 那么就以$DEFAULT作为其值 *
${var:=DEFAULT}     # 如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 *
${var+OTHER}        # 如果var声明了, 那么其值就是$OTHER, 否则就为null字符串
${var:+OTHER}       # 如果var被设置了, 那么其值就是$OTHER, 否则就为null字符串
${var?ERR_MSG}      # 如果var没被声明, 那么就打印$ERR_MSG *
${var:?ERR_MSG}     # 如果var没被设置, 那么就打印$ERR_MSG *
${!varprefix*}      # 匹配之前所有以varprefix开头进行声明的变量
${!varprefix@}      # 匹配之前所有以varprefix开头进行声明的变量
```

### 字符串操作

```shell
${#string}                          # $string的长度 
${string:position}                  # 在$string中, 从位置$position开始提取子串
${string:position:length}           # 在$string中, 从位置$position开始提取长度为$length的子串     
${string#substring}                 # 从变量$string的开头, 删除最短匹配$substring的子串
${string##substring}                # 从变量$string的开头, 删除最长匹配$substring的子串
${string%substring}                 # 从变量$string的结尾, 删除最短匹配$substring的子串
${string%%substring}                # 从变量$string的结尾, 删除最长匹配$substring的子串   
${string/substring/replacement}     # 使用$replacement, 来代替第一个匹配的$substring
${string//substring/replacement}    # 使用$replacement, 代替所有匹配的$substring
${string/#substring/replacement}    # 如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
${string/%substring/replacement}    # 如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
```

比较常用的是利用字符串操作，处理掉文件的后缀，前缀等。例如在我的一个ffmpeg批量转换的脚本中，对raw文件批量转为bmp文件，就可以这样写

```shell
for file in ./*.raw
do
    # 去除开头的./
    filename=${file:2}

    ffmpeg -vcodec rawvideo -f rawvideo -pix_fmt rgb32 -s 1280x720 -i ${filename} -f image2 -vframes 1 -vcodec bmp output/${filename/%.raw/.bmp}
    # 出错停止
    if [ $? -ne 0 ];then
        exit
    fi
done
```

## 常用指令

shell脚本其实更是一个方便批量处理的工具，许多信息也可以通过linux的指令来完成。

### sed 指令

[sed 指令详解](https://wangchujiang.com/linux-command/c/sed.html)

sed是一个非常强大的字符处理工具，可以搭配正则等使用，功能非常强大，用来自动编写文件，对文件反复操作，编写转换程序等。像本文的打印帮助信息就用到了sed。sed的使用非常广泛，有许多奇妙的用法，我也还没有完全掌握，需要不断学习。在[这个文章](https://www.runoob.com/linux/linux-comm-sed.html)中有许多案例可以帮助理解。

> 想要深入学习sed，正则表达式的知识是必不可少的，之后我也打算写一篇关于正则表达式的博客，这里暂且按下不表。

sed 命令

```shell
a\ # 在当前行下面插入文本。
i\ # 在当前行上面插入文本。
c\ # 把选定的行改为新的文本。
d # 删除，删除选择的行。
D # 删除模板块的第一行。
s # 替换指定字符
h # 拷贝模板块的内容到内存中的缓冲区。
H # 追加模板块的内容到内存中的缓冲区。
g # 获得内存缓冲区的内容，并替代当前模板块中的文本。
G # 获得内存缓冲区的内容，并追加到当前模板块文本的后面。
l # 列表不能打印字符的清单。
n # 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。
N # 追加下一个输入行到模板块后面并在二者间嵌入一个新行，改变当前行号码。
p # 打印模板块的行。
P # (大写) 打印模板块的第一行。
q # 退出Sed。
b lable # 分支到脚本中带有标记的地方，如果分支不存在则分支到脚本的末尾。
r file # 从file中读行。
t label # if分支，从最后一行开始，条件一旦满足或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾。
T label # 错误分支，从最后一行开始，一旦发生错误或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾。
w file # 写并追加模板块到file末尾。  
W file # 写并追加模板块的第一行到file末尾。  
! # 表示后面的命令对所有没有被选定的行发生作用。  
= # 打印当前行号码。  
# # 把注释扩展到下一个换行符以前。  
```

sed 替换标记

```shell
g # 表示行内全面替换。  
p # 表示打印行。  
w # 表示把行写入一个文件。  
x # 表示互换模板块中的文本和缓冲区中的文本。  
y # 表示把一个字符翻译为另外的字符（但是不用于正则表达式）
\1 # 子串匹配标记
& # 已匹配字符串标记
```

sed 元字符集

```shell
^ # 匹配行开始，如：/^sed/匹配所有以sed开头的行。
$ # 匹配行结束，如：/sed$/匹配所有以sed结尾的行。
. # 匹配一个非换行符的任意字符，如：/s.d/匹配s后接一个任意字符，最后是d。
* # 匹配0个或多个字符，如：/*sed/匹配所有模板是一个或多个空格后紧跟sed的行。
[] # 匹配一个指定范围内的字符，如/[sS]ed/匹配sed和Sed。  
[^] # 匹配一个不在指定范围内的字符，如：/[^A-RT-Z]ed/匹配不包含A-R和T-Z的一个字母开头，紧跟ed的行。
\(..\) # 匹配子串，保存匹配的字符，如s/\(love\)able/\1rs，loveable被替换成lovers。
& # 保存搜索字符用来替换其他字符，如s/love/ **&** /，love这成 **love** 。
\< # 匹配单词的开始，如:/\<love/匹配包含以love开头的单词的行。
\> # 匹配单词的结束，如/love\>/匹配包含以love结尾的单词的行。
x\{m\} # 重复字符x，m次，如：/0\{5\}/匹配包含5个0的行。
x\{m,\} # 重复字符x，至少m次，如：/0\{5,\}/匹配至少有5个0的行。
x\{m,n\} # 重复字符x，至少m次，不多于n次，如：/0\{5,10\}/匹配5~10个0的行。  
```


### trap 命令

trap指令是一个非常强大的指令，用来指定接受到信号之后的动作，或者在脚本被中断之后执行清理动作，在嵌入式的rcS中就有用到，使用也相对简单，具体看[该博客](https://wangchujiang.com/linux-command/c/trap.html)即可。

### 其他

* 获取操作类型: `uanme -s`
