<font size = 4>

### 前序
这些东西并不是专业的, 只是笔者在使用zsh过程中的一些记录, 很多都是摘抄的[官方](https://zsh.sourceforge.io/Guide/zshguide.html). 个人提取了认为重要的地方. 而且并不是将原文中相关的介绍作详细的深读, 可能只是结合平时场景来举一些常见的案例.

<br/>


### 一些名词映射
|名词|映射|
|:-|:-|
|标准输入|stdout|
|标准输出|stdout|
|标准错误|stderr|
|命令行|command-line, cmd-line, cmdline|
|系统调用|system call, sys-call,syscall|

### 选项
- 选项是让表现出来的行为像特定的shell[^ann-option-0], 都是bool值, 即可以打开或关闭. 通常是大写字母以下划线分开.
- 选项所涉及到的操作有:
    1. 判断
    2. 开启
    3. 关闭

```bash
[[ -o xxx ]]    # 开启了
[[ ! -o noxxx ]]# 开启了, noxxx表示未开启, !:表示取反, 所以意义是:开启了
setopt      xxx     # 手动开启
unsetopt    xxx     # 手动关闭
```




### shell的原理
shell本身就是C程序, 它主要的功能是接收标准输入后将处理结果反馈到标准输出或错误上. 这个过程中很大可能发生系统调用. 这解释的有点专业了[^ann-shell-0]. 说白了就是在命令行键入命令, 然后shell接收后进行处理, 最后将处理的结果打印到屏幕上. 

> 实际上shell的工作原理远比这要复杂的多. 笔者有一门笔记深读了 <font color = yellow>Unix环境高级编程</font>[^ann-shell-1], 这里面从系统的角度解释了shell的工作原理. 所以zsh也是这样. 后面会以zsh来表示shell, 它也是当前笔者的环境.


### 外部命令

zsh接收标准输入[^ann-cmd-0]后, 内部通过识别算法会将所谓的命令分为2类: 
1. 内部命令
2. 外部命令

这里的内外部适合从编程角度来解释: 所谓内部命令(<font color = yellow>也称为内置同住</font>) 就是shell自身的命令, 它不会fork[^ann-cmd-1]出子进程去处理, 而是自己处理, 然后直接将处理结果输出到标准输出. 所以整个过程的前台交互[^ann-cmd-2]程序始终是zsh, 一般来说同样的功能若有程序文件和内置命令时, 内置命令的效率要更高.

对于外部命令, zsh一定会先去找命令的可执行文件, 如在cmdline键入`date`后, zsh先到找`/bin/date`这个可执行文件, 然后调用date这个程序, 最后date向stdout打印时间. zsh是怎么找到这个文件的呢?  答案是通过环境变量PATH. 

PATH变量属于zsh的配置, 其实也是整个Unix标准配置. PATH可以在zsh的脚本文件中指定(`$HOME/.zshrc`). 当在cmiline键入命令时, zsh通过PATH配置的目录中查找, 找到后就启动这个文件创建进程并执行它. 查找的过程是递进的, 即PATH中最前面的目录没有就会找下一个, 找到后就不找了
> 这里其实zsh有优化, 当zsh找到命令后, 如date(`/bin/date`), 会将date所在的目录`/bin`下所有的命令加载到它内部维护的哈希表中, 这有2方面的原因: 1. 加载目录下的文件非常快; 2. 下次找命令将优化从哈希表中查找, 这样比PATH更快. 基于种机制, 会存在这样一个问题: 对于同样一个如ls的命令, 若存在2个目录中(<font color = yellow>允许的</font>)(`/usr/bin/ls`和`/usr/sbin/ls`), 并且在当前的全局哈希表(`对应的是/usr/sbin/ls`), 则会先查找到`/usr/sbin/ls`. 但其实`/usr/bin/ls`下的实现可能更快. 基于这种情况, 可以直接清空哈希表(`rehash`)


### 内置命令也启动新进程
一般情况下内置命令不会开启新的进程, 但zsh提供了一种用法, 运行内置命令时, 在子进程中运行

```bash
# pwd是shell的内置命令, 每次切换目录后, 它都记录当前所在的目录
type pwd
pwd is a shell builtin  # 查看pwd是一个内置命令
pwd
/Users/tierry           # 当前在Home
cd /tmp                 # 切换到 /tmp
pwd
/tmp                    # 当前在 /tmp
cd $HOME
pwd 
/Users/tierry           # 回到了HOME
(cd /tmp)               # 启动子进程中切换到/tmp
pwd                     # 在当前进程中查看
/Users/tierry           # 没有变化
```

### 打印
echo 和 print 是zsh的内置命令. 

---

> print常用选项

|常用选项|说明|
|:-|:-|
|-r|原样输出转义字符, 如`\n`不会解释成换行|
|-n|末尾不要添加换行符, 实际情况是zsh在末尾添加了`%`, 然后等待输入的光标换到了一下行|
|-l|每个参数一行|

---

```bash
%print hello world          # %是zsh默认的等待输入的标记, 光标在它右边
hello world                 # 直接打印 "hello world", 然后换行了
%                           # 等待用度输入

%print -n hello world
hello world%                # 附加-n后, 会在打印后追加%, 然后还是换行了
%                       



%print -r "hello world\n"   # 在字符串手动插入一个换行的转义
hello world\n               # 打印结果直接将 \n 原样输出, 并且还是换行了
%                           # 等待用户输入
```










### 数组下标从0开始计
- 对应的选项是`KSH_ARRYFS`, 默认情况下是关闭的, 即在zsh中数组下标默认从1取, 开启后则默认从0开始取
```bash
# 定义相关的变量 和 函数
arr=(this is a girl)
fn(){
    arr=(this is a pretty girl)
    echo $arr
    if [[ -o KSH_ARRAYS ]]; then
        echo "from 0"
        echo ${arr[0]}          # 如果选项设置了, 必须以这种方式访问数组
        echo ${arr[1]}
    else
        echo "from 1"
        echo $arr[0]            # 没有设置则可以以 $arr[N]访问
        echo $arr[1]
    fi
    setopt KSH_ARRAYS
    echo "true(from 0)"
    echo ${arr[0]}
    echo ${arr[1]}
}

fn                              # 调用函数
this is a pretty girl
from 1                          # 说明选项未设置, 下标从1开始
                                # 第0个是空
this                            # 第1个开始是this
true(from 0)                    # 手动设置后
this                            # 从0开始, 所以是this
is                              # 第1个则是is
```

<br/>

### lazy load func
- 在用到时再加载函数, 步骤:
    1. 在配置文件中`如~/.zshrc`中指定自动加载`autoload myfunc`
    2. 编写脚本文件myfunc(文件不要带后缀)
    3. 将文件所在的目录添加到`fpath`

```bash
# 编写脚本$HOME/.myshell/myfunc文件, 直接在文件中写脚本, 不用在顶部添加 #!/bin/zsh
echo "test auto load func"

# .zshrc中
autoload myfunc
fpath=("$HOME/.myshell" $fpath)     # fpath在开头插入了指定的路径 


# 在zsh启动后, 不会真正加载函数myfunc, 当某一时刻想用时, 直接在命令行键入
myfunc
# 或在脚本文件中直接调用
```


<br/>

### 历史命令的高级操作
- 选择历史命令操作
```bash
echo one two
echo thr fou
!!                  # 明确调用最后1次的命令
echo thr fou        # 会回显, 然后回车输出
thr fou             # 最后1条命令是: "echo thr fou"




echo one two
echo thr fou
!-2                 # 调用倒数第2条命令
echo one two        # 回显
one two             

```

<br/>


- 选择命令参数

```bash
### 选取指定位置的参数
echo one two
echo thr fou        
echo fiv !-2:2      # 调用倒数第2条命令, 并取出它的第2个参数
echo fiv two        # 回显
fiv two             
echo !:1            # !:并未指定调用哪条命令, 但默认指向的是最后1条, 所以打印的是fiv
echo fiv            # 回显
fiv




### 选取所有的参数
echo one two thr fou
echo !:*            # 取最后一条命令的所有参数
echo two thr fou    # 回显
one two thr fou




### 选取最后一个参数
echo one two thr fou
echo !!:$           # 取最后一个参数, 这里用了"!!", 也表示最后一条命令
echo fou            # 回显
fou


### 选取范围内的参数
echo one two thr fou
echo !:2-3          # 取第2到第3个参数
echo two thr        # 回显
two thr
```

<br/>

- 截取参数
    1. `:t`: 一般用来修饰文件路径, 获取文件名(省略路径)
    2. `:h`: 也用来修饰文件名, 获取文件所在的目录
    3. `:r`: 删除后缀
    4. `:l`: 变小写
    5. `:u`: 变大写
    6. `:s/str1/str2`: 将str1第1次(从左)出现的位置替换成str2
    7. `:gs/str1/str2`: 将所有str1替换为str2
    8. `:&/str1/str2`: 同6, 不过是从右向左
    9. `:g&/str1/str2`: 同理8, 全局换


```bash
## 取文件名
echo /users/etc/bin/file.c
echo !:t                        # 这里"!:"本质是一个变量(特殊), 它的值是最后一条命令("echo /users/etc/bin/file.c")
                                # :t的意义: 取变量中最后一个/后所有的内容, 所以是file.c
echo file.c                     # 回显 
file.c



# 取文件路径(有问题)
echo /users/etc/bin/file.c  
echo !:h                        # :h的意义: 取最后一个/前面所有的内容, 所以是 "echo /users/etc/bin"
echo echo /users/etc/bin        # 回显, 注意最前面的是echo, 因为 !:这个特殊变量的内容是"echo /users/etc/bin/file.c", :h取了最后一个/前面所有的内容 
echo /users/etc/bin             # 输出

# ps: 一般对字符串这样操作
path=/users/etc/bin/file.c
echo ${path:h}
/users/etc/bin                  # 这种情况下不会回显, 直接将结果打印出来了




## 大小写
echo hello my name is tierry    
echo !:u                        # 注意 !: 获取到的内容是"echo hello my name is tierry"
echo ECHO HELLO MY NAME IS TIERRY   # 回显
ECHO HELLO MY NAME IS TIERRY    # 变大写, 注意最前面有个echo
echo !:l                        # 变小写 !: ==> "echo ECHO HELLO MY NAME IS TIERRY"
echo echo hello my name is tierry # 变小写, 注意前面又有了echo

# ps: 也可以用作字符串




### 替换
echo hello my name is tierry
!:s/tierry/jerry/               # 注意此刻最前面没有echo, !:的内容是"echo hello my name is tierry", 
echo hello my name is jerry     # 回显
hello my name is jerry          # 最后的输出


# ps: 其他的就不演示了
```

<br/>

> 使用`:h`和`:t`可以将路径中所有的项取出来. 这里写了一个简单的脚本, 必须传递绝对路径
```bash
#!/bin/zsh
## get_file_path.sh, 必须传绝对路径
fn(){
    if [[ $1 == '/' ]]; then    # $1获取的是函数参数
        echo $1
    else
        fn $1:h                 # 因为是从 / 打印, 所以这里先递归调用, 毕竟 $1:h 是不断舍去最右边的
        echo $1:t               # 这里取的是全名, 没有去掉后缀
    fi
    sleep 1
}
fn $1                           # 获取的是脚本文件接收的参数


### 测试
source get_file_pat.sh "/Users/Code/src/vim.c"  # 直接在当前的环境中执行
/
Users
Code
src
vim.c       
```

> 使用递归将给定的 <font color=deeppink>绝对路径</font>切割开打印. 





</font>


[^ann-shell-0]: 笔者解释一个计算机概念时, 会先从编程角度来解释, 然后再给出应用层面的效果来作白话解释.
[^ann-shell-1]: 目前还未完结的, 后面完毕了, 我会将链接贴过来
[^ann-cmd-0]: 即在命令行键入命令, 后面笔者不会特意再指出了, 同理, 当提到标准输出/标准错误时, 表示打印到屏幕
[^ann-cmd-1]: unix中的`few fork exec wait`, zsh在执行外部命令时, 会fork出子进程, 即先克隆自己, 然后找到命令的程序文件创建进程, 这个创建的进程和zsh的关系就是父子关系, zsh自己会等待它完毕, 它们之间是通过管道进行通信的
[^ann-option-0]: zsh本身兼容了很多历史上的shell语法, 所以若是编写跨平台的zsh脚本必须要考虑兼容性, 就是通过zsh提供的选项来判断的
















