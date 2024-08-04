<font size = 5>

# 简介
[fzf](https://github.com/junegunn/fzf)是一个通用的命令行交互式模糊查找器. 具体来说它是一个适用于任何类型列表的交互式过滤程序. 文件,命令历史记录，进程,主机名,书签,git提交等. 它实现了一种模糊匹配算法, 可以快速输入省略字符获取想要的结果.
> 说白了: fzf接收任何能构成列表的记录, 然后给出交互式视图, 用户可以键入关键字筛选出自己想要的结果. fzf是用go语言所写

---

<img src="./.images/fzf-icons/006.png"/>

### 特点
1. 没有依赖性
2. 速度极快
3. 可扩展性极强, 主要体现在与其他程序的配合上

### 基本原理 
fzf接收标准输入的数据, 这些数据的形式类似于列表, fzf收到这些数据后将其显示在交互式窗口中供用户筛选, 用户可以按fzf的模糊规则来查找, 最后被fzf筛选出来到标准输出



# 安装
### homebrew方式
1. `brew install fzf`

2.  在当前的zsh中配置fzf
```bash
# 加载fzf的脚本, 里面主要做了:
## 添加fzf的环境变量
## 添加shell的快捷键
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh  
```
> 还有其他安装方式, 不同的安装方式配置的步骤是不一样的, 可以看[官方说明](https://github.com/junegunn/fzf?tab=readme-ov-file#installation)

### 升级
直接使用对应包管理器, 对于homebrew来说`brew upgrade fzf`就可以了


# 用法
### 无参启动fzf <a id="link-no-arg-start"/>
没有任何参数的fzf将会调用`find`程序将当前目录下所有的文件(<font color = red>递归</font>)展示到交互视图中
> 笔者这里以虚拟机下的linux来演示, 因为该环境下fzf是纯净的.

---

<img src="./.images/fzf-icons/007.png"/>

<img src="./.images/fzf-icons/008.png"/>


### 接收stdin
fzf最大的作用是接收stdin上的数据
```bash
result="";for i in {1..10}; do result=$result"\nid:"$RANDOM; done; echo $result | fzf --layout reverse
```
这是直接在命令行上键入的shell命令. 来分解一下:
1. `result=""`: 定义一个变量为空字符串, 用来存储格式后的列表字符串
2. 循环:
    - 1次: result为`\nidx:随机值`
    - 2次: result为`\nidx:随机值1\nidx:随机值2`
    - 3次: result为`\nidx:随机值1\nidx:随机值2\nidx:随机值3`
    - ..
3. `echo $result`: 输出结果
4. 第3步的结果通过管道将数据传递给了fzf, 这样fzf将结果呈现在交互视图中, 如下图

---

<img src="./.images/fzf-icons/009.png"/>

> 结果中呈现的是`xx/11`, 原因是fzf会默认加一行头部, 这个后面会细说. 最后在交互视图中选中后, fzf会将选中的结果输出到原来的命令行中然后结束交互视图

<br/>

### fzf处理空
fzf接收到空字符串时, 将其打印到标准输出时, 很多情况下没有任何意义. 

```bash
echo "" | fzf           
```
> 键入这句脚本后, fzf也会开启交互视图，但其实列表中没有内容的, 但回车后也会输出换行到屏幕, 这种情况没有意义. 可以使用`--print0`选项, 这样fzf不会打印任何内容到标准标出


### vim场景版本1 
有这样需求: 键入vim时直接在当前目录下选择文件后打开它.

正常情况下在shell中键入vim后, 再按`TAB`shell会给出可编辑的文件让用户选择编辑, 如下图:

---

<img src="./.images/fzf-icons/010.png"/>

> 这也是一种交互, 但太单纯了只能`TAB`往后或`Shift-TAB`往前, 如果有大量的文件去TAB效率将大大降低. 所以可以直接利用fzf. 

```bash
vim $(fzf)

# 或

fzf | xargs -I filename vim filename
```

> 这种形式, 用户直接在列表中选择相应的文件后，fzf将该文件输出到`vim `后后面, 再按回车即可打开vim编辑它

<br/>

### vim场景版本2
上面的版本需要在选择后再点击回车才能进入vim编辑, 现在想实现选中后直接进入vim编辑文件. 首先fzf默认不带附加选项时, 只会将选择的结果输出到标准输出, 所以单纯的使用vim并不能实现该功能

在shell中可以使用`eval`和`command`来实现这个功能

```bash
eval "vim $(fzf)"          

# 或

command vim $(fzf)
```
> 这里借助shell的2个内置命令来完成操作, eval需要的是整体的字符串, cmmand后面可以跟随参数, 它们内部来fork出vim， 将参数传递给vim进程


<br/>


# 列表
### 快捷键
当fzf打开交互列表时, 可以移动的方式:
1. 键中的上下键
2. `Ctrl-K`或`Ctrl-P`:上
3. `Ctrl-J`或`Ctrl-N`:下
4. 对于多选模式(`-m`指定), 使用`TAB`选中, `Shift-TAB`取消选中
5. 也支持鼠标操作, 多选时按住shift然后滚动


<br/>

### 显示模式
默认情况下, 交互视图占据全屏, 并且交互的输入框在最底部

---

<img src="./.images/fzf-icons/011.png"/>

可以通过选项来调整
```bash
## 进入到 /etc
fzf --height 20         # 从光标下开始展示20行的高度,默认情况下文件从交互视图的底部往上列出来
fzf --height 40%        # 从光标下距离底部的窗口高度 * 0.4的高度展开
fzf --height 40% --border # 交互视图周围用线框起来
fzf --height 40% --border --layout reverse # 文件从光标下往底部开始列出来
fzf --hegiht ~40%       # 用~指定最大高度
```
> PS：笔者虚拟机上的fzf版本较低, 这些选项如`~%40`是有有效的


### tmux模式 <a id="link-tmux"/>
很多用户有tmux环境, fzf提供了tmux的支持. 多窗口时可以将交互视图显示在window的上方, 这样方便操作

```bash
# [center top bottom left right][,xxx(size)] [,xx(size)]
# eg: --tmux center 
fzf --tmux center           # 在tmux中央, width,height默认50%
fzf --tmux 80%              # center, WH:80%
fzf --tmux 100%,50%         # center, W:100%, H:50%
fzf --tmux left,40%         # left, W:40%
fzf --tmux left,40%,90%     # left, W:40%, H:90%
fzf --tmux bottom,40%,90%   # bottom, W:40%, H:90%
```
> 当没有在tmux环境下时, 将自动忽略

<br/>



# 搜索规则
### 模式
默认情况下fzf在交互中输入时, 匹配模式是扩展模式, 并不是固定字符串模式. 可以以空格隔开, 递进搜索, 但是无序的

---

<img src="./.images/fzf-icons/013.png"/>

输入`lib`后, 默认是模糊匹配, 只要有`l`,`i`,`b`的都可以匹配出来, 而且是`l-->i-->b`顺序. 但是优先展示的是`lib`, 该图是笔者滚动了交互视图的区域, 排在前面的是`lib`相关的条目

---

<img src="./.images/fzf-icons/012.png"/>

输入`lib git zsh`后, fzf的每一组输入都是模糊匹配的规则, `lib`出现, 但`git`相关的匹配和`lib`没有关系, 可能在穿插在lib匹配结果的中间

<br/>

可以在每组前添加`'`告诉fzf, 该组的匹配按固定字符串模式

--- 

<img src="./.images/fzf-icons/014.png"/>

此时严格按`lib`字符串匹配. 但若下一组还是模糊匹配的, 匹配原则不变

---

<img src="./.images/fzf-icons/015.png"/>

可以发现第1组是严格匹配, 但`li`作为第2组还是模糊匹配, 并且`li`出现的位置并不在`git`后面才出来.


### fzf的匹配算法推导
很明显, 交互时, fzf对用户的键入作时时监听, 每当输入后就会时时的匹配, 并将匹配出的结果刷新在列表中, 下次有字符键入时，以这次的结果作下一次的匹配内容. 所以才会出现乱序. 如:
1. 键入`'git`
2. 筛选出内容有`lib/git/xxx.zsh`
3. 键入`lib`, 则`lib/git/xxx.zsh`也会匹配出来

所以对于下面这个搜索也可以作出解释:

--- 

<img src="./.images/fzf-icons/016.png"/>

> 第1次键入`^l`(以l开头)后匹配出的有`lib/xxxx`, 然后第2组键入`l`, `lib/xxxx`当然可以匹配出


### 匹配规则 

|token|模式|说明|
|:-|:-|:-|
|`lib`|模糊匹配|按`l-->i--b`顺序匹配|
|`'lib`|精确匹配|按`lib`匹配|
|`^lib`|精确匹配|以`lib`开头|
|`.lib$`|精确匹配|以`lib`结尾|
|`!lib`|精确匹配|不包含`.lib`|
|`!^lib`|精确匹配|不以`lib`开头|
|`!.lib$`|精确匹配|不以`.lib`结尾|

可以在调用fzf时使用`-e`或`--exact`, 这样在交互时键入的token都被视为精确匹配模式


### OR运算模式
`lib .framework$ | .a$`表示以`lib`开头结尾是`.a`或`.framework`




<br/>

# 环境变量 
### TTY
[前面](#link-no-arg-start)介绍过, fzf在命令行不接收任何参数启动时, 内部默认调用了系统的find命令来查找当前目录下所有的文件并展示在交互视图中. 这种使用在fzf中被称为`FZF_DEFAULT`. 像`echo hello | fzf`的使用不会触发内部的find. 通过配置相关的环境变量可以改变这一行为, 步骤:
1. `FZF_DEFAULT_COMMAND`: fzf无参启动时, 数据从哪里来(<font color = red>默认是find查找的内容</font>). 
2. `FZF_DEFALUT_OPTS`: 为fzf指定本身的选项.
3. `FZF_DEFAULT_OPTS_FILE`: `FZF_DEFAULT_OPTS`可以使用一个文件来替代, 这样做的目的方便管理默认选项


### `FZF_DEFAULT_COMMAND`
其实它就是告诉fzf列表内容的来源, 这里可以来测试:
1. 有这个变量, 但内容是空
2. 有这个变量, 但内容是其他


```bash
## 在 .zshrc中
export FZF_DEFAULT_COMMAND=""
```

---

<img src="./.images/fzf-icons/017.png"/>

> 可以发现还是调用了find

<br/>

```bash
export FZF_DEFAULT_COMMAND="eoch 'hello world'"
```

--- 

<img src="./.images/fzf-icons/018.png"/>

> 发现该变量里的内容确实改变了无参下fzf的行为


### fzf替换默认的搜索引擎
原理就是指定`FZF_DEFAULT_COMMAND`, 以下是使用`fd`和`ripgrep`来改变fzf的无参默认行为的配置

```bash
export FZF_DEFAULT_COMMAND="fd -H -t f --follow ."
# 或
export FZF_DEFAULT_COMMAND="rg . --files" 
```

> 以上是简单的替换fzf默认的find, fd和rg(<font color = red>ripgrep</font>)搜索速度极快. 


### `FZF_DEFAULT_OPTS`
该环境变量是无参启动fzf时, 默认的选项. 一般情况下会将fzf常用的选项添加进去:
1. `-e`: 搜索时精确匹配
2. `--walker`: fzf间接告诉find搜索的文件类型. <font color = red>若搜索引擎是fd或ripgrep时没有效果</font>, 但为了兼容性最好还是加上:
    - dir: 搜索目录
    - file:搜索文件
    - follow:跟随符号链接
    - hidden:搜索隐藏文件或目录
3. `--walker-skip`: 告诉find不要在什么路径下搜索. 同样搜索引擎是其他时没有效果
    - `--walker-skip=${path1,path2}`
    - 有更多的往后加就行了, 注意格式
4. 布局相关:
    - `--height`
    - `--layout`
    - `--border`
5. 预览视图: 即聚焦切换视图中选项时,要怎么展示. 如有的是图片,有的是pdf, 这就需要区分开用不同的程序去渲染. 后面会有单独的章节介绍预览视图
6. 其他高级的交互选项: 这后面再细说

下面是一个例子<font color = red>注意要写在一行</font>
```bash
export FZF_DEFAULT_OPTS='-e --walker=file,follow,hidden --walker-skip=${.git,node_modules} --height=90%  --tmux bottom,40% --layout=reverse --border=bottom --bind=alt-b:down,alt-h:up --preview="$HOME/.myshell/file-preview.sh {}" --preview-window=right:60%:wrap'

## 多指定了 --tmux选项, 这个配合tmux
### --bind指定了可以使用 Alt+b往下, Alt+h:往上
### --preview将fzf传递的列表值交给 指定的shell脚本去处理, 该脚本的工作就是要区分不同的文件调用不同程序去渲染出预览视图
### --preview-window预览视图布局
```
> 选项中指定了`--walker`相关的是为了兼容, 若fzf的搜索引擎是find(<font color = red>Unix系统原生</font>), 这些选项会被fzf处理并传递到find对应行为的选项. 所以若搜索引擎是`ripgrep`时必须在`FZF_DEFAULT_COMMAND`中指定要对应的选项, 因为fzf本身不知道rg这个程序具体的选项细节.


# 配合shell
### 4个快捷模式
fzf的强大在于可以和其他程序无缝衔接. fzf提供了4个模式:
1. `Ctrl+T`: 获取文件和目录的列表
2. `Alt+C`: 获取目录列表
3. `Ctrl+R`: 获取历史列表
4. `any-cmd [path]**<TAB>`: 任何命令都可以以这种模式调用, 所以这个命令是抽象的, 需要用户自己定义. 它会回调`_fzf_comrun`, 在该函数中用户可以得到的参数:
    - 命令参数`any-cmd $1`
    - 路径`$2`





### `Ctrl+T`
任何命令都可以`cmd Ctrl+T`触发fzf. fzf总会将当前目录下的所有文件(包括目录)列出到交互视图中. 

---

<img src="./.images/fzf-icons/019.png"/>

这个过程调用的是find命令, 并不是`FZF_DEFAULT_COMMAND`. fzf提供`FZF_CTRL_T_COMMAND`和`FZF_CTRL_T_OPTS`来改变这一行为.

```txt
export FZF_CTRL_T_COMMAND=$FZF_DEFAULT_COMMAND
export FZF_CTRL_T_OPTS=$FZF_DEFAULT_OPTS
```

这样列表中的条目只有文件没有目录(<font color = red>前面定义时指定fd或rg搜索的是文件</font>), 所以要重新定义该命令

```bash
export FZF_CTRL_T_COMMAND="fd -H --follow ."           
export FZF_CTRL_T_OPTS=$FZF_DEFAULT_OPTS
```
> 这里指定fd搜索目录和文件, 因为rg只能搜索文件. 但是选项可以和`FZF_DEFAULT_OPTS`共用. 当<font color = red>`FZF_CTRL_T_OPTS`为字符串时,快捷键将被禁用</font>


<br/>



### `Alt+C`
它的过程和`Ctrl+T`是一样的, 但它列表中的选项是目录.  fzf也为其提供了2变量来改变它的行为:
1. `FZF_ALT_C_COMMAND`: 可以指定fd搜索目录
2. `FZF_ALT_C_OPTS`: 指定相关的选项




### `Ctrl+R`
任何命令都可以以`cmd Ctrl+R`来触发fzf, fzf会搜索出用户历史命令的记录(读取的shell):
1. 若再次按下`Ctrl+R`将按时间排序. 
2. 使用`Ctrl+N`或`Ctrl+P`上下移动
3. 使用`Ctrl+/`或`Alt+/`将交换命令的顺序

不像前面, fzf只为其定义了选项的变量`FZF_CTRL_R_OPTION`, 一般在该变量里指定高级选项, 这个到后面学习






### `**<TAB>`
任何命都可以以`cmd **<TAB>`触发fzf, 这个过程和前面的不同. 它更抽象, 前面的2个快捷操作(`Ctrl+T`和`Alt+C`)在设计上要求用户返回目录和文件(<font color = red>逻辑上可以输出任何内容到标准输出,但最好是按原则来</font>). 而这里的设计更抽象, fzf向外界回调函数`_fzf_comprun`, 在该函数里用户可以自定义更丰富的操作

在`**<TAB>`行为中, fzf为vim和cd定义了具体的操作:
> 如`vim **<TAB>`触发后, fzf会向外界调用`_fzf_compgen_path`函数, 并将路径`~/`传递到该函数中. 一般情况下该函数的功能是将获取到的所有文件写入到标准输出. 

<br/>

> 再如`cd **<TAB>`触发后, fzf会向外界调用`_fzf_compgen_dir`函数, 并将当前工作目录传递到该函数, 在这个函数中一般是搜索相关的目录写到标准输出, 因为cd本身只能进行目录. 

<br/>

> 在实现这2个回调函数[^ann-cbk]时并不是要改变预定的行为, 要遵循一个原则:该返回目录的返回目录, 该返回文件的就返回文件, 不要改了默认的行为. 用户在函数内要做的是使用高效的程序去搜索目录和文件. <font color = red>若定义了`_fzf_comprun`则vim和cd回调的是comprun</font>


笔者给出这几个相关函数的定义大致定义过程:

```bash
# 供 vim $HOME**tab 的回调函数, 返回文件
_fzf_compgen_path() {
    # 这里的 . 是通配符, 必须指定表示搜索任何文件, 并不是当前目录
    ### $1是$HOME
   fd -H -t f --follow --exclude={$FZF_IGNORE_SEARCH_PATHS} . $1
}

# 供 cd **tab 的回调函数, 返回目录
_fzf_compgen_dir() {
   fd -H -t d --follow --exclude={$FZF_IGNORE_SEARCH_PATHS} . $1
}
```

<br/>

### `_fzf_comprun`
笔者觉得有必要将这个函数拿出来单独强调. 当用户提供了该回调函数后, `vim\cd **<TAB>`的行为将发生改变, 会直接来到这里. 该函数是所有命令以`cmd **<TAB>`所触发. 这里测试一下不同命令下该函数中的参数列表

```bash
## .zshrc中
_fzf_comprun(){
    echo "" > /tmp/a.log
    for arg in $@; do
        echo $arg >> /tmp/a.log
    done;
    echo "over"     # 必须向标准输出输出内容
}
```

> 笔者直接整理了结果:

```bash
rm /tmp/a.log
vim $HOME**             # vim $HOME**<TAB>

cat /tmp/a.log
vim
-m
-q
--walker
file,dir,follow,hidden
--walker-root=/Users/liubo      # 被fzf回调时, 传递的第1个参数是命令名vim, 其余选项是fzf默认规定的, 但路径位于 `--walker-root`




rm /tmp/a.log
cd $HOME**              # cd $HOME**<TAB>
cd
-q
--walker
dir,follow
--walker-root=/Users/liubo      # 对于cd来选项的个数又不一样, 如只有目录, 其他的像路径也一样



rm /tmp/a.log
echo hello world**      # echo hello world**<TAB> 
-m
-q
world
--walker
file,dir,follow,hidden
--walker-root=.                 # 参数和vim一样, 但是路径却是当前目录("."), 并不是指定的hello
```

<br/>

总结: `cmd **<TAB>`被触发时, fzf会传递命令名(<font color = red>$1</font>)以及默认的一些选项`-m -q --walker`等, 路径位于`--walker-root`中, 并且路径必须是合法的, 即以`/`或`./xx`或`../xx`, 所以一般情况下该函数应该处理成如下这样:

```bash
_fzf_comprun() {
  local command=$1
  shift
    # $@是fzf内部传递出来的选项和值, 用户自己参数没有
  case "$command" in
    cd)           fzf --preview 'tree -C {} | head -200'   "$@" ;;
    export|unset) fzf --preview "eval 'echo \$'{}"         "$@" ;;
    ssh)          fzf --preview 'dig {}'                   "$@" ;;
    *)            fzf --preview 'bat -n --color=always {}' "$@" ;;
  esac
}
```

> 这是官方给出的, 从里面可以看出来, 区分了不同的命令, 但最后都是要执行fzf来唤起交互视图

# 选项
fzf功能强大, 可定制性高, 所以内部实现复杂. 通过`man fzf`可以查看所有的功能介绍. 在手册里对功能分了类,下面笔者将简单介绍一下常用的选项



### 搜索功能
|简|全|说明|
|:-|:-|:-|
|`-x`|`--extended`|扩展模式, 这是默认的|
|`+x`|`--no-extended`|上面取反|
|`-e`|`--exact`|精确匹配|
|`-i`|`--ignore-case`|忽略大小写,要是有大写就匹配大写|
|`+i`|`--no-ignore`|不忽略大小写|
|无|`--algo=type`|模糊算法类别,v2最佳评分算法(默认). v1更改但不保证找到最佳结果|

### 搜索结果

|简|全|说明|
|:-|:-|:-|
|`+s`|`--no-sort`|不要对结果排序|
|无|`--tail=NUM`|最多显示多少条数据到列表中|
|无|`--track`|当视图中的结果列表更新时, 让光标继续选择之前的条目|
|无|`--tac`|对结果数据进行反转,如history使用该选项后,最新的在最上面|


### 交互设置
|简|全|说明|
|:-|:-|:-|
|`-m`|`--multi`|多选模式|
|`+m`|`--no-multi`|上面取反|
|无|`--no-mouse`|禁用鼠标|
|无|`--bind=KEYBINDS`|高级操作,后面有单独的章节|
|无|`--cycle`|列表可以循环滚动|
|无|`--wrap`|[见下](#link-wrap)|
|无|`--wrap-sign`|[见下](#link-wrap)|


<a id="link-wrap"></a>
对于选项`--wrap`以及`--wrap-sign`是当列表中的条目过长时,自动换行. 可以指定换行后显示的换行标记

---

<img src="./.images/fzf-icons/032.png"/>



### 布局
|简|全|说明|
|:-|:-|:-|
|无|`--height=[~]HEIGHT[%]`|交互视图的高度,从鼠标下到窗口的底部这段区域|
|无|`--tmux`|交互视图位于[tmux](#link-tmux)的上层|
|无|`--layout=LAYOUT`|[见下](#link-layout)|
|无|`--border[=BORDER_OPT]`|[见下](#link-border)|
|无|`--border-label`|有上下边框时, 可以在边框上添加标签|
|无|`--border-label-pos`|标签的位置|
|无|`--margin`|[见下](#link-space)|
|无|`--padding`|[见下](#link-space)|
|无|`--info`|[见下](#link-info)|
|无|`--info-command`|[见下](#link-info-command)|
|无|`--separator`|[见下](#link-info-separator)|
|无|`--scrollbar`|[见下](#link-scrollbal)|
|无|`--prompt`|[见下](#link-prompt)|
|无|`--pointer`|[见下](#link-pointer)|
|无|`--marker`|[见下](#link-marker)|
|无|`--marker-multi-line`|[见下](#link-marker-multi)|
|无|`--header`|[见下](#link-header)|
|无|`--header-line`|[见下](#link-header-line)|
|无|`--header-first`|[见下](#link-header-first)|
|无|`--ellipsis`|[见下](#link-ellipsis)|


对于选项`--height`:
1. 若是一个负数, 则交互视图的高度为终端高度送去该值
2. 若指定了`~`时, fzf会自动计算高度, 比如给一个`~100%`时, fzf会根据列表数量来确定高度, 若数量较少,则自动调整合适的高度. 但它使用有以下限制:
    - 不能与以百分比大小给出的顶部,底部边距和填充一起使用
    - 不能使用负值, 即`~-2`
    - 当有多行项目时, 它将找不到正确的尺寸


<a id="link-layout"></a>
对于选项`--layout`, 它表示交互视图从窗口什么地方展示,有3种
1. default: 输入框在最底部, 在输入框上面的是列表视图
2. reverse: 输入框从命令行下开始,下面是列表视图
3. reverse-list: 输入框在窗口底部, 列表视图在命令行下面


<br/>

<a id="link-border"></a>
对于选项`--border`, 它的作用是将交互视图用:
1. rounded: 4周圆角边框
2. sharp: 4周锐角角边框
3. bold: 4周粗线边框
4. double: 4周双线边框
5. horizonal: 有2条边, 一条位于命令行下,一条位于最底部
6. vertical: 有2条边, 一条在视图的左边, 一条在视图的右边
7. top: 只有1条, 位于视图上边
8. left: 同上, 位于左边
9. right: 同上, 位于右边
10. bottom: 同上, 位于下边
11. none:没有边框


<br/>

对于选项`--border-lable`以及`--border-label-pos`:
1. 标签不仅仅是文字, 可以是任何输出到标准输出上的内容
2. 标签总是展示在上或下边框, 所以`--border`指定的模式要包含上下边框
3. pos指定位置的格式:
    - `--border-label-pos=[N[:top|bottom]]`
    - 当不指定top或bottom, 则默认在上边框上展示标签
    - N为正表示从左边起第N个字符开始展示
    - N为负表示标签距离窗口右边多少个字符

---
```bash
## 在上边的center展示
fzf --layout default --border --border-label=$(echo '~~~~~~~~~~~傻逼~~~~~~~~~' | lolcat -f -S 444)

## 在下边距离窗口右边3个字符
fzf --layout default --border --border-label=$(echo '~~~~~~~~~~~傻逼~~~~~~~~~' | lolcat -f -S 444) --border-label-pos=-3:bottom
```
<img src="./.images/fzf-icons/020.png"/>

<img src="./.images/fzf-icons/021.png"/>




<a id="link-space"></a>
对于选项`--margin`以及`--padding`表示交互视图相关的间距:

---

<img src="./.images/fzf-icons/022.png"/>

> 使用格式:
>
> 1. `--margin 5%`: 交互视图距离窗口的上下左右为一样的间距
> 2. `--margin 5%,%10`: 交互视图距离窗口的上下为50%,  左右为10%
> 3. `--margin 5%,%20, 5%`, 交互视图距离窗口的上为5%, 左右为20%, 下为5%  w  
> 4. `--margin 5%,%20, 5%, 10%`, 交互视图距离窗口的上为5%, 右20%, 下5%, 左10%
> 
> padding是一样的用法


<br/>


<a id="link-info"></a>
对于选项`--info`, 它指的是:

<img src="./.images/fzf-icons/023.png"/>


它的选项值就是指定它的位置:
1. default: 默认, 位于输入框下, 左边开始
2. right: 同上, 但从右边开始
3. hidden: 不展示
4. inline: 和输入框在同一行, 左边
5. `inline:prefix`: 和输入框在同一行, 但自定义了前缀
6. inline-right: 和输入框在同一行, 但在右边
7. `inline-right:prefix`; 同上,在右边, 并自定义了前缀


----


<img src="./.images/fzf-icons/024.png"/>
<img src="./.images/fzf-icons/025.png"/>
<img src="./.images/fzf-icons/026.png"/>
<img src="./.images/fzf-icons/027.png"/>

---


<a id="link-info-command"></a>
对于选项`--info-command`的作用是由外界指定info的内容渲染. 

```bash
fzf --border --info-command='echo "tierry@@:$FZF_POS#$FZF_INFO"'
```

---

<img src="./.images/fzf-icons/028.png"/>

> 其中选项的值必须是字符串, 在这里是一个echo命令. 必须用单引号. fzf内部会启动子进程变成shell调用echo, `$FZF_POS`表示info的起始位置, `$FZF_INFO`有原始的info信息(`64/64`)



<a id="link-info-separator"></a>
对于选项`--separator`, 它表示info这条线的样式, 默认是`-`
```bash
fzf --info-command 'echo "tierry@@:$FZF_POS#$FZF_INFO"' --separator=~
```

---

<img src="./.images/fzf-icons/029.png"/>


<a id="link-scrollbar"></a>
对于选项`--scrollbar`表示滚动条. 在交互视图中可能出现在列表页及详情页, 对应指定的格式为`--scrollbar=char1[char2]`,其中`char1`表示列表页, `char2`表示详情页. 只能指定ASCII字符, 不能Unicode

```bash
fzf --info-command 'echo "tierry@@:$FZF_POS#$FZF_INFO"' --separator=~ --scrollbar='&{'
```

---

<img src="./.images/fzf-icons/030.png"/>


<a id="link-prompt"></a>
对于选项`--prompt`表示输入框左边的字符
```bash
fzf --layout reverse --border --prompt=&
```

---

<img src="./.images/fzf-icons/031.png"/>



<a id="link-pointer"></a>
对于选项`--pointer`表示列表项前面的字符, 默认是`|`

---

<img src="./.images/fzf-icons/033.png"/>



<a id="link-marker"></a>
对于选项`--marker`表示列表在选中时前面前面显示的字符, 默认是`|`

---

<img src="./.images/fzf-icons/034.png"/>



<a id="link-marker-multi"></a>
对于选项`--marker-multi-line`表示列表中条目换行时, 要显示的字符

```bash
\ls -al | fzf --wrap           
\ls -al | fzf --wrap --wrap-sign="->"
```

---

<img src="./.images/fzf-icons/035.png"/>

<img src="./.images/fzf-icons/036.png"/>


<a id="link-header"></a>
对于选项`--header`是以列表添加一个标题, 这个标题所在的行是粘性的, 即在滚动时, 它不动. 它的值是一个字符串, 当然可以是一上带颜色的终端字符串

```bash
\ls -al | fzf --header="$(echo "I'm tirrey" | lolcat -f -S 72)"
```

---

<img src="./.images/fzf-icons/037.png"/>


<a id="link-header-line"></a>
对于选项`--header-line`指定header以及下面有多少行也是粘性的

```bash
\ls -al | fzf --header="$(echo "I'm tirrey" | lolcat -f -S 72)" --header-lines=3
```

---

<img src="./.images/fzf-icons/038.png"/>


<a id="link-header-first"></a>
对于选项`--header-line`header这一行显示在输入框的上方

```bash
\ls -al | fzf --header="$(echo "I'm tirrey" | lolcat -f -S 72)" --header-lines=3 --header-first
```

---

<img src="./.images/fzf-icons/039.png"/> w

<a id="link-ellipsis"></a>
对于选项`--ellipsis`列表中的每一行过长时, 若不换行将截断. 该选项值的作用是截断的部分用它显示, 默认是`..`

```bash
\ls -al | fzf 

\ls -al | fzf --ellipsis="-->more content"
```


---

<img src="./.images/fzf-icons/040.png"/>

<img src="./.images/fzf-icons/041.png"/>



### 渲染
这里只介绍`--color`选项. 前面小节布局中出现的模块视图(<font color = red>如info, prompt, header等</font>)包括下个小节的预览视图都可以输出颜色. 颜色选项的使用格式:
```txt
--color=[BASE_SCHEME][,COLOR_NAME[:ANSI_COLOR][:ANSI_ATTRIBUTES]]..
```

|`BASE_CHEME`|说明|
|:-|:-|
|dark|终端256种dark模式|
|light|终端256种light模式|
|16|终端16颜色|
|bw|禁用颜色, 也等于选项`--no-color`|


<br/>

|`COLOR_NAMES`|说明|
|:-|:-|
|fg|文字, 列表中的文字和预览中文字颜色|
|`selected-fg`|多选时, 列表选中条目的文字颜色|
|`preview-fg`|预览视图中的文字颜色|
|bg|整个交互视图的背景颜色, 包括预览部分|
|`select-bg`|多选时, 列表选中条目的背景颜色|
|`preview-bg`|预览视图的背景颜色|
|hl|[见下](#link-color-hl)|
|`current-fg`或`fg+`|[见下](#link-color-hl)|
|`current-bg`或`bg+`|[见下](#link-color-hl)|
|info|对应部分的颜色设置|
|border|同上|
|scrollbar|同上|
|`perview-border`|同上|
|`perview-scrollbar`|同上|
|separator|同上|
|label|同上|
|`perview-label`|同上|
|prompt|同上|
|pointer|同上|
|marker|同上|
|header|同上|


<br/>


ANSI是终端下的颜色, 若是16,表示只支持16种颜色. 若是256表示支持255位颜色,数字代表的颜色小伙伴可以到网络上查询, 这个是统一标准的. 还有可能是支持RGB模式的, 这就是平常用的`#ff000000`

ATTRIBUTES只用于前景, 即文字. 如下:

|ATTRIBUTES|说明|
|:-|:-|
|regular|正常|
|bold|加粗|
|underline|下划线|
|italic|斜体|
|strikethrough|删除线|




<a id="link-color-hl"></a>
对于选项`hl`,`fg+`及`bg+`, 效果如下:

```bash
\ls -al | fzf --color='light,hl:72,fg+:9'
\ls -al | fzf --color='light,hl:72,fg+:9,bg+:13'
```

---

<img src="./.images/fzf-icons/042.png"/>
<img src="./.images/fzf-icons/043.png"/>




# 预览
### 预览选项
|选项|说明|
|:-|:-|
|`--preview`|[语法](#link-preview)|
|`--preview-label`|预览视图的标签文字, 原理同`--border--label`|
|`--preview-label-pos`|同`--border-lable-pos`
|`--preview-window`|[见下](#link-preview-widow)|


<a id="link-preview"></a>
### 预览语法
当列表中选中某个条目时, 用户可以指定与该条目相关的信息到单独的预览视图中, 如展示文件内容

---

<img src="./.images/fzf-icons/044.png"/>

预览选项的值是一个shell命令, 上图中简单的使用了`cat {}`. 这个命令字符串被fzf解析:
1. `{}`: 表示获取选中条目的整体内容
2. `{+}`: 同上
3. `{+1}`: 选中条目以空白字符切割, 从左边第1个字段
4. `{-1}`: 选中条目以空白字符切割, 从右边第1个字段
5. `{f}`: 下面的小节
6. `{q}`: 引用输入框中的字符串
7. `{n}`: 引用当前选中列表的序号
8. `{n3..n5}`: 第3到第5列
9. `{n3..}`: 第3列到最后
10. `{..3}`: 第1列到第3列
11. `..`: 所有的列

---

<img src="./.images/fzf-icons/045.png"/>

```bash
# 定义一个变量, 4行, 每一行用空格将目录和文件隔开
# 使用fzf展示的时候是4行
paths="a.dir a.txt\nb.dir b.txt\nc.dir c.txt\nd.dir d.txt"


echo $paths | fzf --preview='cat {}'               # {} 表示整行记录, 即 cat "a.dir a.txt"
                                                   # error: 没有"a.dir a.txt"的文件jjjjj

echo $paths | fzf --preview='cat {1}'              # {1} 获取的是空格隔开的第1上字段, 即"a.dir"
                                                   # cat a.dir, error: 它是一个目录

echo $paths | fzf --preview='cat {1}/{2}'          # 会被解析成 "a.dir/a.txt"

echo $paths | fzf --preview='cat {-1}/{-1}'        # {-1}右边起第1个字段, 所以 {-2}/{-1}被解析成 a.dir/a.txt 
```


### delimiter选项
这个选项用来指定列表中条目的分割符. 前面小节介绍的, 默认情况下fzf对当前选中条目内容以空白切割, 然后使用`{N}`来引用切割后的字段. 可以手动指定切割字符.
```bash
rg "^#" -v --no-line-number /etc/passwd | fzf --wrap --delimiter : --preview 'echo {-3}'
```
读取`/etc/passwd`文件中所有匹配的内容, 然后列表展示出来, 以`:`切割, 在预览视图中打印倒数第3个字段

<img src="./.images/fzf-icons/047.png"/>


### `{f}`的场景
当用户在列表中进行选择时, 默认情况下preview中获取条目内容的方式是`{}`. 还有一种获取条目完整内容方式是`{f}`, fzf会将条目的内容写入到一个临时文件中, `{f}`就是对这个文件的引用. 下面写一个测试脚本来看下`{f}`的本质
1. 编写脚本`get_tmp_file.sh`内部打开获取的参数
2. 赋于对应的执行权限

```bash
## get_tmp_filename.sh

#!/bin/zsh
echo "arg-0":$0
echo "arg-1":$1

chmod +x get_tmp_filename.sh

## 测试
seq 10 | fzf --preview="./get_tmp_filename.sh {f}"
```
---

<img src="./.images/fzf-icons/046.gif"/>


### 不同文件预览的处理
当用fzf列出文件时, 可以对不同类型的文件指定不同的程序去渲染. 比如图片使用图片程序去预览,pdf使用pdf程序去预览. 原理就是通过设置`--preview`实现

```bash
## 使用单独的脚本或函数来统一处理
fzf --preview="./show_file.sh {}"           


## 脚本文件
#!/bin/zsh
file=$1
echo $file
if [[ -d ${file} ]]; then
 eza -a -l -B --time-style="+%%m-%d %H:%M" --git --color=always --icons=always $file
elif [[ $(file --mime ${file}) =~ binary ]]; then
 if [[ $(file --mime ${file}) =~ "image/" ]]; then
     columns=$(tput cols)
     columns=$((${columns} / 2))
     catimg -w ${columns} "${file}"
 else
     echo "${file} is a binary file" && false
 fi
else
 bat --theme="Solarized (dark)" --style=numbers --color=always "${file}" 2> /dev/null |
head -100
fi
```
> 上述只是一个简单的脚本, 当是一个目录时, 使用`eza`来展示该目录下的文件; 接着判断是不是图片, 并不是根据文件的后缀, 而是通过MIME来判断, 若是图片则使用`catimg`这个程序来展示图片. 其他若是二进制文件则提示是一个二进制. 其他所有的情况调用bat预览


<a id="link-preview-window"></a>
### `--preview-window`
该选项是对预览视图的布局作出控制, 基本格式
```txt
--preview-window=
    [POSITION] [,SIZE[%]]                      
    [,border-BORDER_OPT] 
    [,[no]wrap] 
    [,[no]follow] 
    [,[no]cycle]
    [,[no]hidden]
    [,+SCROLL[OFFSETS][/DENOM]]
    [,~HEADER_LINES]
    [,default]
    [,<SIZE_THRESHOLD(ALTERNATIVE_LAYOUT)]
```

选项里所的指定都可以省略. 

1. POSITION确定预览界面在窗口中的位置. 有4个:up; down; left; right. `SIZE`表示预览视图的大小, 以fzf整个交互视图的宽度来算. 当size为0时, 不会渲染预览视图, 但是fzf还是会在后台计算
2. boder边框样式, 和`--border`选项值差不多
3. wrap预览内容是否可换行
4. follow设置后, 将自动滚动到最底部
5. cycle表示滚动条是否可以循环滚动, 即滚动一最后再往下滚动又从第1行开始
6. hidden不显示预览视图
7. `+SCROLL[OFFSETS][/DENOM]`整体表示预览视图初始化时滚动条应该在第几行. `+SCROLL`有2种值:
    1. 具体的整数, 表示第几行
    2. 对当前选中条目切割后指定字段的引用, 必须保证字段是整数, 即解析出来也表示第几行
    3. `OFFSETS`表示以SCROLL为基准往上下偏移, 上为负,下为正
    4. `/DENOM`将预览的行号往预览窗口的中部放
8. `~HEADER_LINES`表示悬浮在顶部固定的行数
9. default将之前所有项设置为默认
10. `<SIZE_SHRESHOLD(ALTERNATIVE_LAYOUT)`,可以设置一个阈值, 当达到这一阈值时,将按`ALTERNATIVE_LAYOUT`布局, 不常用

举一个手册中的案例: 用git查找关键字为`VC`相关的文件记录.

```bash
git grep --line-number 'VC' | fzf --preview --delimiter : 'echo {1}' --preview-window '+{2}-5'
```


### 关键字预览
很多时候需要根据关键字搜索并展示该关键字相邻的几行信息. 这里面涉及到的功能:
1. 根据关键字查找对应的文件
2. 定位到文件的具体行
3. 展示该行相邻的几行

前2个功能可以使用ripgrep, 展示可以使用bat. 笔者写一个简单的脚本(`test.sh`)来实现

```bash
#!/bin/zsh
np=$[ $2 - 3 ]
nn=$[ $2 + 3 ]
bat -l Objective-C --color always $1 -r $np:$nn


rg -F $1 ./ | fzf --delimiter : --preview "/tmp/test.sh {1} {2} $1"
```

---

<img src="./.images/fzf-icons/048.gif"/>


### 脚本
|简|全|说明|
|:-|:-|:-|
|`-q`|`--query`|启动时就传递搜索关键字|
|`-1`|`--select-1`|只有一个结果时,直接选中在命令行,不会有交互视图|
|`-0`|`--exit-0`|没有结果时,直接退出|
|`-f`|`--filter`|直接变成命令行搜索|
|无|`--print-query`|第1行打印查询的字符串|
|无|`--expect`|[见下](#link-expect)|
|无|`--read0`|[见下](#link-read0)|


<br/>

<a id="link-expect"></a>
对于选项`--expect`, 它的作用是指定键位完成Enter同样的操作, 但它比Enter会多输出键位名到第1去. 当使用该选项时, Enter键触发后, 也会在第1行打印一个空行. 
```txt
--expect="ctrl-a,ctrl-v,@,f1,f2" --epxect="alt-s"
```
> 指定多个选项时,会取并集

它的优先级要高于`--bind`


<br/>

<a id="link-read0"></a>
对于选项`--read0`表示接收的列表数据中不以换行为准, 以`\0`字符为准

```bash
echo "hello\0world" | fzf --read0
```

---

<img src="./.images/fzf-icons/049.png"/>




# 绑定
### 简介
fzf提供了事件绑定机制以便实现动态运行时行为. 
```txt
fzf --bind="key1:action,key2:action,eveent:action"
```

### 热键KEY
键盘上所有的键都可以绑.
1. `ctrl-xx`: xx为键盘上可见字符
2. `ctrl-alt-[a-z]`
3. `alt-xx`:同ctrl
4. enter
5. space
6. backspace
6. tab
6. `shift-tab`

> 其他的可以看手册 


### 事件EVENT
|事件|说明|
|:-|:-|
|start|fzf启动时会调用, 一般要指定`--sync`|
|load|标准输入流可用并且列表数据渲染完成时调用|
|resize|终端尺寸发生变化调用|
|result|过滤搜索完成, 结果已经准备好时调用|
|change|键入搜索关键字发生变化就调用|
|focus|光标从当前行切换到其他行触发|
|one|当匹配的结果只有一个时触发,`one:accept`和`--select-1`一样, 但它是动态的|
|zero|当匹配没有结果时触发,`zero:abort`和`--exit-0`一样|
|jump|跳转到指定的目标后触发|
|click-header|鼠标点击head触发, 会传递`FZF_CLICK_HEADER_LINE`和`FZF_CLICK_HEADER_COLUMN`|



<font color = red>start</font>: 在fzf初始化时会调用一次, 一般情况下指定该事件的action时会同时指定`--sync`.

```bash
seq 1000 | fzf -m --sync --bind start:last+select-all
```
> 整个命令表示生成1000个序列, 启动fzf后默认滚动到最后, 并全部选中. 若未指定`--sync`则可能最后并未出现last和select-all的效果. 因为fzf内部的事件和输入流之间是异步的, 可能fzy启动后在调用start事件时, 标准输入流还没有数据. 所以指定`--sync`指定让start的事件等待标准输入可用

<br/>

<font color = red>load</font>: 当标准输入流可用并且列表初始化处理完成时触发. 也会触发一次

```bash
# 使用load时不需要添加 --sync, 因为此时输入流已可用
seq 1000 | fzf -m --bind "load:last+select-all"

## 
seq 1000 | fzf -m --prompt "loading$" --bind load:change-pormpt:tierry$
```

---

<img src="./.images/fzf-icons/050.gif"/>


### ACTION
这个具体看手册, 有很多.



### 执行外部程序
可以设置快捷键来执行外部程序

```bash
fzf --bind 'f1:execute(less -f {}),ctrl-y:execute-silent(echo {} | pbcopy)+abort'
```
> 功能是按下f1后以less命令打开文件. 摁下`ctrl-y`对当前选中的条目做复制并退出, `execute-silent`是后台静默模式.

### 变成新的进程
上面执行外部程序时,fzf并未结束,而是在新的窗口执行了less, 而`become`则将当前fzf变成指定的程序
```bash
fzf --bind enter:become(vim {})
```
> 该命令在选中条目后,自动用vim打开, 并结束fzf

### 刷新当前条目
reload事件可以在不重启fzf的情况下, 直接在当前窗口中对数据刷新. 
```txt
--bind change:reload:rg {q}
```
> 功能是在键入过程中, 不停的调用rg查关键字查询, 并将查询的结果更新在列表中


# 笔者的配置
### 基本配置
笔者将fzf的配置放到单独的脚本文件中, 以便维护 
```bash
# 以下2个环境变量分类过滤了相关的目录, 这些目录里大部分是库文件, 平时搜索最好过滤不搜
#  第1类是系统的目录: 这些目录一般是操作系统或包管理软件, 不需要搜索, 会耗费大量的时间
#  第2类是用户的目录: 这3个目录里面含有大量的第3方库文件
export OS_LIB_PATHS="Applications,Library,.cache,.cargo,.cocoapods,.gem,.local,.mygit,.oh-my-zsh,.rustup,.rvm,.vim,.Trash"
export TIERRY_STUDY_PATHS="Pods,linux_code,linux_core"


# 加载fzf的脚本文件初始化fzf. 
# 1. 将fzf的可执行文件添加到环境变量
# 2. 适配zsh
[ -f $HOME/.fzf.zsh ] && source $HOME/.fzf.zsh      

# 过滤上述2类目录
export FZF_IGNORE_SEARCH_PATHS=".git,node_modules,${OS_LIB_PATHS},${TIERRY_STUDY_PATHS}"
# 过滤上述第1类目录
export TIERRY_IGNORE_SEARCH_PATHS=".git,node_modules,${OS_LIB_PATHS}"


# fzf搜索引擎替换
### 搜索引擎fd配置
    # -H: 让fd搜索隐藏目录和文件
    # -t: 只要文件
    # --follow: 跟随软链接
    # --exclude: 让fd在搜索时过滤的目录(因为这些目录中的文件太多了)
    # PS: 可以在FZF_DEFAULT_OPTS中指定过滤目录, 但没有用, 因为fzf默认调用的是系统的find, 它不知道fd去除过滤的目录选项, 注意不能换行书写
#export FZF_DEFAULT_COMMAND="fd -H -t f --follow --exclude={${FZF_IGNORE_SEARCH_PATHS}}"
export FD_DEFAULT_COMMAND="fd -H --follow --exclude={${FZF_IGNORE_SEARCH_PATHS}}" # 使用fd搜索目录和文件, 过滤2类目录



export RIPGREP_CONFIG_PATH=$HOME/.myshell/rg.config                         # 导出rg配置文件, rg本身会用到该变量
export FZF_RG_ALL_GLOB="--glob='!"'{'${FZF_IGNORE_SEARCH_PATHS}'}'"'"       # rg的glob选项, 这里过滤了2类目录
export FZF_RG_FIR_GLOB="--glob='!"'{'${TIERRY_IGNORE_SEARCH_PATHS}'}'"'"    # rg的glob选项, 这里过滤了第1类目录


export TIERRY_RG_GLOB=${FZF_RG_FIR_GLOB}                                    # 自定义一个用户级别的glob, 过滤第1类目录. 因为可能去 linux下的目录搜索东西
export TIERRY_RG_COMMAND="rg --files ${TIERRY_RG_GLOB}"                     # 对应的用户级别的命令

export FZF_DEFAULT_COMMAND="rg --files ${FZF_RG_ALL_GLOB}"                  # 无参启动时的fzf命令, 过滤2类目录 

# 全局的选项, 这些选项是fzf需要的
WALKER_TYPE="--walker=file,follow,hidden"                                   # 当搜索引擎是find时, 要搜索文件,软链接,隐藏文件
WALKER_FILTER_DIR="--walker-skip=\"${FZF_IGNORE_SEARCH_PATHS}\""            # 当搜索引擎是find时, 要过滤的2类目录
FZF_DEFAULT_OPTS_HEIGHT="--height 90%"                                      # fzf的交互视图高度, 从光标下到窗口底部的90%
FZF_DEFAULT_OPTS_LAYOUT="--layout reverse"                                  # 交互视图的输入框在命令行下方, 再下方是列表视图
FZF_DEFAULT_OPTS_TMUX="--tmux center,80%"                                   # tmux模式时, 交互窗口在最上层, 大小为窗口的80% 
FZF_DEFAULT_OPTS_BORDER="--border double"                                   # 整个交互视图用双线包裹
FZF_DEFUALT_OPTS_PREVIEW='--preview "$HOME/.myshell/file-preview.sh {}"'    # 使用 file-preview来处理各种文件的打开
FZF_DEFUALT_OPTS_PREVIEW_WINDOW="--preview-window 'right:60%:wrap,~1'"      # 预览视图的布局

FZF_DEFAULT_OPTS_BORDER_POS="--border-label-pos -5"                         # 标签栏的位置, 上方从右边起数5个字符
FZF_DEFAULT_OPTS_INFO="--info inline-right:\<"                              # 提示信息,在输入框一行右边
FZF_DEFAULT_OPTS_ELLIPSIS="--ellipsis ' <more...>'"                         # 列表中的条目显示不够时, 显示更多
FZF_DEFALUT_OPTS_COLOR="--color 'dark,hl:72:underline,fg+:9:bold,bg+:16,border:202,preview-label:202'"          # 匹配字的绿高亮并加下划线(hl:72); 选中条目的文字红(fg+:9); 选中条目的背景暗红(bg+:16); 边框颜色黄红(border:202)

FZF_DEFAULT_OPTS_OTHER_EX="$FZF_DEFAULT_OPTS_BORDER_POS $FZF_DEFAULT_OPTS_INFO $FZF_DEFAULT_OPTS_ELLIPSIS  $FZF_DEFALUT_OPTS_COLOR"

FZF_DEFAULT_OPTS_BIND="--bind 'alt-b:down,alt-h:up,start:change-border-label($(echo tierry | lolcat -f -S 444))+change-header($(echo "搜索结果\n" | lolcat -f -S 72))'"
# 高级语法, 使用transform时,引用{}要加\{}, transform里意味着可以写shell相关的命令
FZF_DEFAULT_OPTS_BIND_PREVIEW_LABLE='--bind "focus:transform-preview-label:echo $(basename \{1})"'
# PS: 这里指定了--walker选项, 它提供了fzf本身要过滤的目录, 这样的目的在于fzf使用默认的find时, 也可以过滤上述2类目录, 加快搜索.--preview: 预览视图, 通过脚本内部统一处理,目前Alacritty下展示图片有问题
export FZF_DEFAULT_OPTS="-e --exit-0 --select-1 $WALKER_TYPE $WALKER_FILTER_DIR $FZF_DEFAULT_OPTS_HEIGHT $FZF_DEFAULT_OPTS_LAYOUT $FZF_DEFAULT_OPTS_TMUX  $FZF_DEFAULT_OPTS_BORDER  $FZF_DEFUALT_OPTS_PREVIEW  $FZF_DEFUALT_OPTS_PREVIEW_WINDOW $FZF_DEFAULT_OPTS_BIND $FZF_DEFAULT_OPTS_OTHER_EX  $FZF_DEFAULT_OPTS_BIND_PREVIEW_LABLE"



# 摁下Ctrl+T后, 展示目录和文件
export FZF_CTRL_T_COMMAND=$FD_DEFAULT_COMMAND;
export FZF_CTRL_T_OPTS=$FZF_DEFAULT_OPTS;

# 摁下Alt+C后, 展示目录
export FZF_ALT_C_COMMAND="fd -H -t d --follow --exclude={${FZF_IGNORE_SEARCH_PATHS}}"
export FZF_ALT_C_OPTS=$FZF_DEFAULT_OPTS;

export FZF_COMPLETION_TRIGGER='\'                                           # **事件触发改为 "\"


# 以下2个函数是"vim \TAB" 或 "cd \TAB" 后触发
# 过滤2类目录
function _fzf_compgen_path() {
    # 必须写了命令, fzf回调出来的
    eval "rg --files ${FZF_RG_ALL_GLOB} $1"  
}

function _fzf_compgen_dir() {
    fd -H -t d --follow --exclude={$FZF_IGNORE_SEARCH_PATHS} . $1
}

function gof() {
    eval "$FZF_DEFAULT_COMMAND $1 | fzf $FZF_DEFAULT_OPTS  --bind 'enter:become(vim {})'"
}

## 暂时未实现
function god() {
    #eval "$FD_DEFAULT_COMMAND -t d . $1 | fzf $FZF_DEFAULT_OPTS --bind 'enter:become(cd {+1})+abort'"
}
```

上面的效果如下图
1. `vim \TAB`
2. `cd \TAB`
3. `cd Ctrl+R`
4. `gof $HOME`

---

<img src="./.images/fzf-icons/051.png"/>
<img src="./.images/fzf-icons/052.png"/>
<img src="./.images/fzf-icons/053.png"/>
<img src="./.images/fzf-icons/054.png"/>



### 高级rg用法
笔者实现了2个功能:
1. 根据关键字搜索, 高亮展示, 并定位预览
2. 在1的基础上来回切换fzf的模糊搜索

```bash
# fzf query frome rg
#
# 函数最多传递2参数
#   $1: 正则表达式, 若没有则".*", 相当于任何内容都搜索
#   $2: 搜索路径,   若没有则".",  相当于在当前目录下搜索
#
# rg部分:
#   主要是构建列表数据.
#   1. 第1次启动直接根据关键词$Q来构建fzf的列表数据
#   2. 后续键入关键词后根据该词搜索{q}, 若为空则搜索全部内容, 构造fzf的列表数据
#   PS: 列表数据格式"filename:linenumber:content(高亮)"
#
# fzf部分:
#   op_disabled:    屏蔽fzf本身的模糊搜索,因为我们要手动调用rg来刷新
#   op_noselect:    覆盖全局中的 select-1选项
#   op_query:       主要用于第1次, 启动后在输入框显示搜索的关键词. 后续显示用户时时键入的关键词
#   op_color:       启用颜色处理(--ansi),不然列表条目不能被rg渲染, 并且预览视图没有数据. 重新设置选中的背景和文字颜色
#   op_delimiter:   指定":"切割
#   op_preview:     覆盖全局的preview配置, 主要多了高亮{2}(预览视图中匹配的行高亮)
#   op_window:      在全局配置的基础上改成上下结构,并定位
#   bind_change:    在输入框内容变化时, 由rg重新在当前目录下搜索, 并刷新列表
#   bind_vim:       绑定回车以vim打开文件, 并定位到该行
#   PS: 这里要使用rg的高亮, 必须关闭fzf的颜色(--ansi), 然后设置(fg+:-1,表示还原)
function fqr(){
    Q=${1:-"\".*\""}
    dir_path=${2:-.}
    cmd_lst="rg ${FZF_RG_ALL_GLOB} --color always --line-number --no-heading"

    op_query='--query $Q'
    op_disable='--disabled'
    op_noselect='--no-select-1'
    op_delimiter='--delimiter ":"'
    bind_vim='--bind "enter:become(vim {1} +{2})"'
    op_color='--ansi --color "fg+:-1:bold,bg+:240"'
    op_window='--preview-window "bottom,+{2}+3/3,~3"'
    bind_change='--bind "change:reload($HOME/.myshell/tierry-fqc-reload.sh $cmd_lst $dir_path {q})"'
    op_preview='--preview "bat  --theme=\"Solarized (dark)\" --style numbers --color always {1}  2> \"/dev/null\"  --highlight-line {2}"'
    cmd_fzf="fzf $op_disable $op_noselect $op_query $op_color $op_delimiter $op_preview $op_window $bind_change $bind_vim"

    eval "$cmd_lst $Q $dir_path | $cmd_fzf"
}

---

<img src="./.images/fzf-icons/055.png"/>


# fzf query toggle(rg,fzf)
#
# 使用ctrl-t来回切换rg和fzf的搜索. 
# 记录了2种状态的关键字, 切换后会还原, fzf的搜索是基于rg给出的列表
# rg模式下可以清空关键字重新搜索给出新的列表
function fqt(){
    Q=${1:-"\".*\""}
    dir_path=${2:-.}
    cmd_lst="rg ${FZF_RG_ALL_GLOB} --color always --line-number --no-heading" 


    rg_record_file="/tmp/.$$-rg.txt"
    fzf_record_file="/tmp/.$$-fzf.txt"
     
    ## 选项以及选项的值是给fzf的, 直接使用单引号, 不要转义
    op_query='--query $Q'
    op_disable='--disabled'
    op_noselect='--no-select-1'
    op_delimiter='--delimiter ":"'
    op_prompt='--prompt "M(rg)> "'
    bind_vim='--bind "enter:become(vim {1} +{2})"'
    op_color='--ansi --color "fg+:-1:bold,bg+:240"'
    op_window='--preview-window "bottom,+{2}+3/3,~3"'
    bind_change='--bind "change:reload($HOME/.myshell/tierry-fqc-reload.sh $cmd_lst $dir_path {q})"'

    ## 注意要解析出 $rg_record_file, 所以必须在双引号内
    cond_rg='[[ $FZF_PROMPT =~ "rg" ]]'
    unbind='echo "unbind(change)+change-prompt(M(fzf)> )+enable-search+transform-query:echo \{q} > '"$rg_record_file"'; cat '"$fzf_record_file\""
    rebind='echo "rebind(change)+change-prompt(M(rg)> )+disable-search+transform-query:echo \{q} > '"$fzf_record_file"'; cat '"$rg_record_file\""
    bind_toggle="--bind 'ctrl-t:transform:$cond_rg && $unbind || $rebind'" 

    op_preview='--preview "bat  --theme=\"Solarized (dark)\" --style numbers --color always {1}  2> \"/dev/null\"  --highlight-line {2}"'
    cmd_fzf="fzf $op_disable $op_noselect $op_query $op_color $op_delimiter $op_prompt $op_preview $op_window $bind_change $bind_toggle $bind_vim"

    eval "$cmd_lst $Q $dir_path | $cmd_fzf"
}
```

---


<img src="./.images/fzf-icons/056.png"/>
<img src="./.images/fzf-icons/057.png"/>
<img src="./.images/fzf-icons/058.png"/>
<img src="./.images/fzf-icons/059.png"/>


















