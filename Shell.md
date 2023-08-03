


# 基本概念与使用


## 基础

Shell使用空格进行分割，如果想输入作为字符串一部分的空格的话，需要写在引号内或者使用转义符号`\`。

执行的命令默认来自于`$PATH`。下面列出的`$PATH`由冒号分割，每一部分都是存储命令的路径：

```bash
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

可以使用`which`查看程序名代表的程序的路径：

```bash
missing:~$ which echo
/bin/echo
```

也可以不用`$PATH`，直接指定路径：

```bash
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

- 标记: 一般以`-`开头。
- 选项: 带有值的标记。

![Linux](Linux.md#权限)

`ls -l /path`可以更具体地查看路径下的所有文件，列出的格式如下：

```bash
drwxr-xr-x 1 fileOwner  users  4096 Jun 15  2019 fileName
```

`drwxr-xr-x`s是1+3+3+3的组合，表示：`d`意为这是一个目录，`rwx`为文件所有者的权限，`r-x`为用户组的权限，`r-x`为其他所有用户的权限。

>`/bin`下所有程序的其他所有用户的权限中都包含x。


## 文件间连接（重定向）

- 重定向输入流： `< file` 
- 重定向输出流： `> file`
- `>>`: 向文件中追加内容。

重定向时，**1代表正确**的标准输出（1可以省略），**2代表错误**的标准输出。例如：`grep "test" file 1 > one 2 > two`会把正确输出和错误输出打印到one文件和two文件。

|符号|说明|
|---|---|
|0|标准输入|
|1|标准输出|
|2|错误输出|
|/dev/null|Linux中的一个特殊文件，写入该文件的内容都将被丢弃|

- `&n`: 表示n输出通道。如1>&2 会让正确返回值传递给2输出通道，其中&2表示2输出通道。
- `&>`:将标准输出和错误输出都重定向到同一个目标。

echo会打印参数，配合重定向输出可以把参数放入一个文件中。

```bash
cat < hello.txt > hello2.txt
```

`|`: **管道**，将一个程序的输出和另一个程序的输入连接起来。

![Linux](Linux.md#xargs)

下面的脚本会找到所有`.html`后缀的文件，将它们压缩为一个压缩包：

```bash
find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf html.zip
```

## sudo

su: super user 或 root。

>有一件事情是您必须作为根用户才能做的，那就是向 `sysfs` 文件写入内容。系统被挂载在 `/sys` 下，`sysfs` 文件则暴露了一些内核（kernel）参数。 因此，您不需要借助任何专用的工具，就可以轻松地在运行期间配置系统内核。

`|`、`>`、和 `<` 是通过 shell 执行的，而不是被各个程序单独执行。因此，重定向输出到系统文件时，即使加了sudo也会报无权限错误。**应当配合tee来实现系统文件的写入**。如：

```bash
#echo把3无脑输出
#在此之前，系统会尝试打开brightness并写入，此时并没有用sudo变成root，因此无权限
sudo echo 3 > brightness

#正确的写入方式
#tee此时是一个root运行的程序，有权打开brightness并写入
echo 3 | sudo tee brightness
```

# 工作流管理与优化

## 任务控制

shell使用unix提供的**信号**机制来进行进程间通信。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。信号是一种**软中断**。

>`kill`命令本质是传信号

用于**结束**进程的信号（看程序是否自觉）：
- `Ctrl-C`会发送`SIGINT`信号
- `Ctrl-\`会发送`SIGQUIT`信号
- `kill -TERM <PID>`会发送`SIGTERM`信号

`SIGKILL` 是一个特殊的信号，它**不能被进程捕获并且它会马上结束该进程**。不过这样做会有一些副作用，例如留下**孤儿进程**。

`SIGSTOP`会让进程**暂停**。在终端中，键入 `Ctrl-Z` 会让shell发送`SIGTSTP`信号。`SIGTSTP`是 Terminal Stop 的缩写（即`terminal`版本的SIGSTOP）。

`fg`和`bg`命令分别可以让暂停的任务前台继续和后台继续。

如果想让当前在运行的**前台任务转后台**，则可以`Ctrl-Z`暂停让后进行`bg`。

`jobs`可以列出当前终端会话中尚未完成的全部任务。会列出pid来引用任务，或者可以用`百分号＋job列出的任务编号`来引用任务，或者使用`$!`来引用上一个任务。

命令中的`&`后缀可以让命令直接在后台运行，但其输出默认是标准输出。

**关闭终端时**会发送`SIGHUP`信号来关闭后台进程。

>注意，后台的进程仍然是您的终端进程的子进程，一旦您关闭终端（会发送另外一个信号`SIGHUP`），这些后台的进程也会终止。为了防止这种情况发生，您可以使用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (一个用来忽略 `SIGHUP` 的封装) 来运行程序。针对已经运行的程序，可以使用`disown` 。除此之外，您可以使用终端多路复用器来实现

```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs
```

## 终端多路复用 tmux

现在最流行的终端多路器是 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。`tmux` 是一个高度可定制的工具，您可以使用相关快捷键创建多个标签页并在它们间导航。

- **会话** - 每个会话都是一个独立的工作区，其中包含一个或多个窗口
    - `tmux` 开始一个新的会话
    - `tmux new -s NAME` 以指定名称开始一个新的会话
    - `tmux ls` 列出当前所有会话
    - 在 `tmux` 中输入 `<C-b> d` ，将当前会话分离
    - `tmux a` 重新连接最后一个会话。您也可以通过 `-t` 来指定具体的会话
- **窗口** - 相当于编辑器或是浏览器中的标签页，从视觉上将一个会话分割为多个部分
    - `<C-b> c` 创建一个新的窗口，使用 `<C-d>`关闭
    - `<C-b> N` 跳转到第 _N_ 个窗口，注意每个窗口都是有编号的
    - `<C-b> p` 切换到前一个窗口
    - `<C-b> n` 切换到下一个窗口
    - `<C-b> ,` 重命名当前窗口
    - `<C-b> w` 列出当前所有窗口
- **面板** - 像 vim 中的分屏一样，面板使我们可以在一个屏幕里显示多个 shell
    - `<C-b> "` 水平分割
    - `<C-b> %` 垂直分割
    - `<C-b> <方向>` 切换到指定方向的面板，<方向> 指的是键盘上的方向键
    - `<C-b> z` 切换当前面板的缩放
    - `<C-b> [` 开始往回卷动屏幕。您可以按下空格键来开始选择，回车键复制选中的部分
    - `<C-b> <空格>` 在不同的面板排布间切换

扩展阅读： [这里](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 是一份 `tmux` 快速入门教程， [而这一篇](http://linuxcommand.org/lc3_adv_termmux.php) 文章则更加详细，它包含了 `screen` 命令。您也许想要掌握 [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html) 命令，因为在大多数 UNIX 系统中都默认安装有该程序。


## 别名 alias

别名是对长命令的缩写，从而可以用简短的字符替代一长串的命令。

```bash
#alias_name="command_to_alias arg1 arg2" 是alias的唯一参数
alias alias_name="command_to_alias arg1 arg2"
```

```bash
# 创建常用命令的缩写
alias ll="ls -lh"

# 能够少输入很多
alias gs="git status"
alias gc="git commit"
alias v="vim"

# 手误打错命令也没关系
alias sl=ls

# 重新定义一些命令行的默认行为
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# 别名可以组合使用
alias la="ls -A"
alias lla="la -l"

# 在忽略某个别名
\ls
# 或者禁用别名
unalias la

# 获取别名的定义
alias ll
# 会打印 ll='ls -lh'
```

>在默认情况下 shell 并不会保存别名。为了让别名持续生效，需要将配置放进 shell 的启动文件里，像是`.bashrc` 或 `.zshrc`。

## 配置文件（Dotfiles）


很多程序的配置都是通过纯文本格式的被称作**点文件**的配置文件来完成的（它们的文件名以`.`开头）

对于 `bash`来说，在大多数系统下，可以通过编辑`.bashrc`或`.bash_profile`来进行配置。

>实际上，很多程序都要求您在 shell 的配置文件中包含一行类似 `export PATH="$PATH:/path/to/program/bin"` 的命令，这样才能确保这些程序能够被 shell 找到。

[Linux 配置文件](Linux.md#配置文件)

## 远端设备 ssh

### 基础

ssh=**Secure Shell** （安全shell）

以用户名foo登录服务器bar.mit.edu：

```bash
ssh foo@bar.mit.edu
```

在`ssh foo@bar.mit.edu`后面直接加命令就可以直接执行命令。 `ssh foobar@server ls | grep PATTERN` 会在本地查询远端 `ls` 的输出； `ls | ssh foobar@server grep PATTERN` 会在远端对本地 `ls` 输出的结果进行查询。

### 认证机制

使用 [`ssh-keygen`](http://man7.org/linux/man-pages/man1/ssh-keygen.1.html) 命令可以生成一对密钥：

```
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

您可以为密钥设置密码，防止有人持有您的私钥并使用它访问您的服务器。您可以使用 [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) 或 [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) ，这样就不需要每次都输入该密码了。

`ssh` 会查询 `.ssh/authorized_keys` 来确认那些用户可以被允许登录。可以通过下面的命令将一个公钥拷贝到这里：

```bash
cat .ssh/id_ed25519 | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

如果支持 `ssh-copy-id` 的话，可以使用下面这种更简单的解决方案：

```bash
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

### 通过 SSH 复制文件

使用 ssh 复制文件有很多方法：

- `ssh+tee`, 最简单的方法是执行 `ssh` 命令，然后通过这样的方法利用标准输入实现 `cat localfile | ssh remote_server tee serverfile`。
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) ：当需要拷贝大量的文件或目录时，使用`scp` 命令则更加方便，因为它可以方便的遍历相关路径。语法如下：`scp path/to/local_file emote_host:path/to/remote_file`。
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 对 `scp` 进行了改进，它可以检测本地和远端的文件以防止重复拷贝。它还可以提供一些诸如符号连接、权限管理等精心打磨的功能。甚至还可以基于 `--partial`标记实现断点续传。`rsync` 的语法和`scp`类似。

### 端口转发

分为本地端口转发和远程端口转发。

常见的情景是使用本地端口转发，即远端设备上的服务监听一个端口，而您希望在本地设备上的一个端口建立连接并转发到远程端口上。例如，我们在远端服务器上运行Jupyter notebook并监听`8888`端口。 然后，建立从本地端口`9999`的转发，使用`ssh -L 9999:localhost:8888 foobar@remote_server`。这样只需要访问本地的`localhost:9999`即可。

### ssh配置

为参数创建一个别名是个好想法，我们可以这样做：

```bash
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

不过，更好的方法是使用 `~/.ssh/config`.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# 在配置文件中也可以使用通配符
Host *.mit.edu
    User foobaz
```

这么做的好处是，使用 `~/.ssh/config` 文件来创建别名，类似 `scp`、`rsync`和`mosh`的这些命令都可以读取这个配置并将设置转换为对应的命令行选项。

注意，`~/.ssh/config` 文件也可以被当作配置文件，而且一般情况下也是可以被导入其他配置文件的。不过，如果您将其公开到互联网上，那么其他人都将会看到您的服务器地址、用户名、开放端口等等。这些信息可能会帮助到那些企图攻击您系统的黑客，所以请务必三思。

服务器侧的配置通常放在 `/etc/ssh/sshd_config`。您可以在这里配置免密认证、修改 ssh 端口、开启 X11 转发等等。 您也可以为每个用户单独指定配置。

### 杂项

连接远程服务器的一个常见痛点是遇到由关机、休眠或网络环境变化导致的掉线。如果连接的延迟很高也很让人讨厌。[Mosh](https://mosh.org/)（即 mobile shell ）对 ssh 进行了改进，它允许连接漫游、间歇连接及智能本地回显。

有时将一个远端文件夹挂载到本地会比较方便， [sshfs](https://github.com/libfuse/sshfs) 可以将远端服务器上的一个文件夹挂载到本地，然后您就可以使用本地的编辑器了。

# 命令工具常见共同功能

- 大部分工具支持 `--help` 或者类似的标志参数（flag）来显示它们的简略用法。
- 会造成不可撤回操作的工具一般会提供“空运行”（dry run）标志参数，这样用户可以确认工具真实运行时会进行的操作。这些工具通常也会有“交互式”（interactive）标志参数，在执行每个不可撤回的操作前提示用户确认。
- `--version` 或者 `-V` 标志参数可以让工具显示它的版本信息（对于提交软件问题报告非常重要）。
- 基本所有的工具支持使用 `--verbose` 或者 `-v` 标志参数来输出详细的运行信息。多次使用这个标志参数，比如 `-vvv`，可以让工具输出更详细的信息（经常用于调试）。同样，很多工具支持 `--quiet` 标志参数来抑制除错误提示之外的其他输出。
- 大多数工具中，使用 `-` 代替输入或者输出文件名意味着工具将从标准输入（standard input）获取所需内容，或者向标准输出（standard output）输出结果。
- 会造成破坏性结果的工具一般默认进行非递归的操作，但是支持使用“递归”（recursive）标志函数（通常是 `-r`）。
- 有的时候你可能需要向工具传入一个 _看上去_ 像标志参数的普通参数，比如：
    - 使用 `rm` 删除一个叫 `-r` 的文件；
    - 在通过一个程序运行另一个程序的时候（`ssh machine foo`），向内层的程序（`foo`）传递一个标志参数。
    这时候你可以使用特殊参数 `--` 让某个程序 _停止处理_ `--` 后面出现的标志参数以及选项（以 `-` 开头的内容）：
- `rm -- -r` 会让 `rm` 将 `-r` 当作文件名；
- `ssh machine --for-ssh -- foo --for-foo` 的 `--` 会让 `ssh` 知道 `--for-foo` 不是 `ssh` 的标志参数。

# shell脚本

shell函数和脚本有如下一些不同点：
- 函数只能与shell使用相同的语言，脚本可以使用任意语言。因此在脚本中包含 `shebang` 是很重要的。
- 函数仅在定义时被加载，脚本会在每次被执行时加载。这让函数的加载比脚本略快一些，但每次修改函数定义，都要重新加载一次。
- 函数会在当前的shell环境中执行，脚本会在单独的进程中执行。因此，函数可以对环境变量进行更改，比如改变当前工作目录，脚本则不行。脚本需要使用`export`将环境变量导出，并将值传递给环境变量。
- 与其他程序语言一样，函数可以提高代码模块性、代码复用性并创建清晰性的结构。shell脚本中往往也会包含它们自己的函数定义。

## shebang 解释伴随行

脚本本质可以使用任意语言，此时就需要shebang来指定。

`#!`符号就是shebang，写在第一行，指定解释器。

`#! bin/bash`就是指定用bash（shell）解释命令；`#!/usr/local/bin/python`则指定用python，例如：

```bash
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

也可以使用`env`，它会通过`$PATH`去寻找解释器，如可以这样指定Python：`#!/usr/bin/env python`。

## 语法

`foo=bar`将变量foo赋值为bar。`$foo`对变量取值。

脚本语句中不能随便加空格，否则会被解释为多个参数或命令调用。此时就需要加引号。**单引号**`'`为原义字符串，其中的变量不会被转义；而**双引号**`"`定义的字符串会将变量值进行替换。

```bash
foo=bar
echo '$foo'
# 打印 $foo
echo "$foo"
# 打印 bar

#跳转到$HOME/marco_history.log里面记录的路径
cd "$(cat "$HOME/marco_history.log")"
```

特殊变量：
- `$0` - 脚本名
- `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值
- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
- `$_` - 上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。
- `$pwd` - 当前路径

>命令通常使用 `STDOUT`来返回输出值，使用`STDERR` 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 

>返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生。

同一行的多个命令可以用 `;` 分隔。

程序 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。

命令替换（command substitution）: 以变量的形式获取一个命令的输出，`$(CMD)`会调用命令CMD，然后用返回结果替换`$(CMD)`。例如，如果执行 `for file in $(ls)` ，shell首先将调用`ls` ，然后遍历得到的这些返回值。

进程替换（process substitution）: 类似命令替换，但获取输出的时候通过临时文件而非标准输入输出。`<(CMD)`会调用命令CMD，将其返回值输出到一个临时文件中，然后再用临时文件路径替换`$(CMD)`。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。

if后面如果用**单方括号**的话，大于小于号等要转义，且不能使用与、非。建议平时使用双方括号，即使不能兼容`sh`。

**通配符**：`*`匹配任意个字符，`?`匹配一个字符，`{xxx,yyy}`匹配其中一个串。如`*{.py,.sh}`匹配所有以.py或.sh结尾的串。


>编写 `bash` 脚本有时候会很别扭和反直觉。例如 [shellcheck](https://github.com/koalaman/shellcheck) 这样的工具可以帮助你定位sh/bash脚本中的错误。

下面这段脚本会遍历我们提供的参数，使用`grep`搜索字符串`foobar`，如果没有找到，则将其作为注释追加到文件中。

```bash
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

下面这段脚本会不断运行`./buggy.sh`并将其输出保存到`out.log`，直到其返回值非0（即运行失败），最后out.log保存的就是错误运行的那次的日志。

```bash
#!/usr/bin/env bash
 count=0
 echo > out.log

 while true
 do
     ./buggy.sh &>> out.log
     if [[ $? -ne 0 ]]; then
         cat out.log
         echo "failed after $count times"
         break
     fi
     ((count++))

 done
```



# shell工具

## 查看手册 man

>如果您想要知道关于程序参数、输入输出的信息，亦或是想要了解它们的工作方式，请试试 `man` 这个程序。它会接受一个程序名作为参数，然后将它的文档（manual用户手册）展现给您。使用 `q` 可以退出该程序。

## 查找文件 find

find会**递归**地查找符合要求的文件：

```bash
# 查找所有名称为src的文件夹
find . -name src -type d
# 查找所有文件夹路径中包含test的python文件
find . -path '*/test/*.py' -type f
# 查找前一天修改的所有文件
find . -mtime -1
# 查找所有大小在500k至10M的tar.gz文件
find . -size +500k -size -10M -name '*.tar.gz'
#以PATTERN模式匹配
find -name '*PATTERN*'
#以PATTERN模式匹配，不区分大小写
find -iname '*PATTERN*'
```

find后面能直接接操作命令，表示对符合要求的文件进行对应操作：

```bash
# 删除全部扩展名为.tmp 的文件
find . -name '*.tmp' -exec rm {} \;
# 查找全部的 PNG 文件并将其转换为 JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

>[`fd`](https://github.com/sharkdp/fd) 是一个更简单、更快速、更友好的程序，它可以用来作为`find`的替代品。它有很多不错的默认设置，例如输出着色、默认支持正则匹配、支持unicode并且我认为它的语法更符合直觉。以模式`PATTERN` 搜索的语法是 `fd PATTERN`。
 大多数人都认为 `find` 和 `fd` 已经很好用了，但是有的人可能想知道，我们是不是可以有更高效的方法，例如不要每次都搜索文件而是通过编译索引或建立数据库的方式来实现更加快速地搜索。
 这就要靠 [`locate`](https://man7.org/linux/man-pages/man1/locate.1.html) 了。 `locate` 使用一个由 [`updatedb`](https://man7.org/linux/man-pages/man1/updatedb.1.html)负责更新的数据库，在大多数系统中 `updatedb` 都会通过 [`cron`](https://man7.org/linux/man-pages/man8/cron.8.html) 每日更新。这便需要我们在速度和时效性之间作出权衡。而且，`find` 和类似的工具可以通过别的属性比如文件大小、修改时间或是权限来查找文件，`locate`则只能通过文件名。 [这里](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)有一个更详细的对比。

## 查找内容 grep

grep是用于对输入文本进行匹配的通用工具。

- `-C`: context，其后再传入一个数字x，会输出匹配结果前后五行。
- `-v`: invert，反选，把不符合要求的给输出出来。
- `-R`: recursive，会递归地查找。

`命令 | grep 关键字`即可对命令的输出进行筛选。

ripgrep (`rg`) 是一个替代品，例子如下：

```bash
# 查找所有使用了 requests 库的文件
rg -t py 'import requests'
# 查找所有没有写 shebang 的文件（包含隐藏文件）
rg -u --files-without-match "^#!"
# 查找所有的foo字符串，并打印其之后的5行
rg foo -A 5
# 打印匹配的统计信息（匹配的行和文件的数量）
rg --stats PATTERN
```


## 查找shell命令 history

history会显示出shell命令的历史记录。如果想搜索历史记录，可以将其传给grep进行模式查找，如`history | grep abc`会找出所有包含abc串的命令。


>对于大多数的shell来说，您可以使用 `Ctrl+R` 对命令历史记录进行回溯搜索。敲 `Ctrl+R` 后您可以输入子串来进行匹配，查找历史命令行。
 反复按下就会在所有搜索结果中循环。在 [zsh](https://github.com/zsh-users/zsh-history-substring-search) 中，使用方向键上或下也可以完成这项工作。
 `Ctrl+R` 可以配合 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 使用。`fzf` 是一个通用对模糊查找工具，它可以和很多命令一起使用。这里我们可以对历史命令进行模糊查找并将结果以赏心悦目的格式输出。


## 文件夹导航

>之前对所有操作我们都默认一个前提，即您已经位于想要执行命令的目录下，但是如何才能高效地在目录 间随意切换呢？有很多简便的方法可以做到，比如设置alias，使用 [ln -s](https://man7.org/linux/man-pages/man1/ln.1.html) 创建符号连接等。而开发者们已经想到了很多更为精妙的解决方案。
 由于本课程的目的是尽可能对你的日常习惯进行优化。因此，我们可以使用[`fasd`](https://github.com/clvv/fasd)和 [autojump](https://github.com/wting/autojump) 这两个工具来查找最常用或最近使用的文件和目录。
 Fasd 基于 [_frecency_](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm) 对文件和文件排序，也就是说它会同时针对频率（_frequency_）和时效（_recency_）进行排序。默认情况下，`fasd`使用命令 `z` 帮助我们快速切换到最常访问的目录。例如， 如果您经常访问`/home/user/files/cool_project` 目录，那么可以直接使用 `z cool` 跳转到该目录。对于 autojump，则使用`j cool`代替即可。
 还有一些更复杂的工具可以用来概览目录结构，例如 [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) 或更加完整的文件管理器，例如 [`nnn`](https://github.com/jarun/nnn) 或 [`ranger`](https://github.com/ranger/ranger)。

## 流编辑器 sed

sed是一种流编辑器，可以传简短的命令去修改文件内容，这在修改远程文件的时候比较好用。可以使用正则表达式匹配内容。

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

## 另一种编辑器 awk

本质是变成语言，但很擅长编辑文本。可以替代grep和sed。


# 杂项






























