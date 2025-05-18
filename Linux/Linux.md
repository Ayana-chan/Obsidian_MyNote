
# 基础知识


## 根路径意义

[文件系统，层次结构标准（Filesystem, Hierarchy Standard）](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

- `/bin` - 基本命令二进制文件
- `/sbin` - 基本的系统二进制文件，通常是root运行的
- `/dev` - 设备文件，通常是硬件设备接口文件
- `/etc` - 主机特定的系统配置文件
- `/home` - 系统用户的主目录
- `/lib` - 系统软件通用库
- `/opt` - 可选的应用软件
- `/sys` - 包含系统的信息和配置([第一堂课](https://missing-semester-cn.github.io/2020/course-shell/)介绍的)
- `/tmp` - 临时文件( `/var/tmp` ) 通常重启时删除
- `/usr/` - 只读的用户数据
    - `/usr/bin` - 非必须的命令二进制文件
    - `/usr/sbin` - 非必须的系统二进制文件，通常是由root运行的
    - `/usr/local/bin` - 用户编译程序的二进制文件
- `/var` -变量文件 像日志或缓存

## 设备

在Linux系统中，每个设备都被当成一个文件来对待，放在`/dev/`中。

|设备|设备在Linux内的文件名|
|---|---|
|SCSI/SATA/USB硬盘机|/dev/sd[a-p]|
|USB闪存盘|/dev/sd[a-p] （与SATA相同）|
|VirtI/O界面|/dev/vd[a-p] （用于虚拟机内）|
|软盘机|/dev/fd[0-7]|
|打印机|/dev/lp[0-2] （25针打印机） /dev/usb/lp[0-15] （USB 接口）|
|鼠标|/dev/input/mouse[0-15] （通用） /dev/psaux （PS/2界面） /dev/mouse （当前鼠标）|
|CDROM/DVDROM|/dev/scd[0-1] （通用） /dev/sr[0-1] （通用，CentOS 较常见） /dev/cdrom （当前 CDROM）|
|磁带机|/dev/ht0 （IDE 界面） /dev/st0 （SATA/SCSI界面） /dev/tape （当前磁带）|
|IDE硬盘机|/dev/hd[a-d] （旧式系统才有）|


磁盘分区后，会在对应文件名后面加数字（1、2、...）。
## 权限

- r: read查看
- w: write修改
- x: execute执行

一般以`rwx`序列来表示权限。`-`表示不具备某权限，如`r-x`表示无权执行。


# 指令

[Linux 命令大全 | 菜鸟教程](https://www.runoob.com/linux/linux-command-manual.html)

- 文件管理 - `cd`, `pwd`, `mkdir`, `rmdir`, `ls`, `cp`, `rm`, `mv`, `tar`
- 文件检索 - `cat`, `more`, `less`, `head`, `tail`, `file`, `find`
- 输入输出控制 - 重定向, 管道, `tee`, `xargs`
- 文本处理 - `vim`, `grep`, `awk`, `sed`, `sort`, `wc`, `uniq`, `cut`, `tr`
- 正则表达式
- 系统监控 - `jobs`, `ps`, `top`, `kill`, `free`, `dmesg`, `lsof`

## 基础
### ls

列出当前目录下的所有文件。

- 更具体的列表: `-l` （等价于`ll`命令）
- 所有文件（包括隐藏文件）: `-a`
- 文件打印以人类可以理解的格式输出 (例如，使用454M 而不是 454279954): `-h`
- 文件以最近访问顺序排序: `-t`
- 以彩色文本显示输出结果: `--color=auto`  

### tree

以树的形式显示指定路径下的所有文件及子文件。
### mkdir

创建目录。

### touch

创建文件。

`touch {1..10}.html`会创建`1.html`~`10.html`十个文件。

### mv

文件移动. 可以用于重命名. 可以:
- 移动文件到目录中
- 移动目录到目录中
- 移动文件成为另一个文件

- **-b**: 当目标文件或目录存在时，在执行覆盖前，会为其创建一个备份。
- **-i**: 如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。
- **-f**: 如果指定移动的源目录或文件与目标的目录或文件同名，不会询问，直接覆盖旧文件。
- **-n**: 不要覆盖任何已存在的文件或目录。
- **-u**：当源文件比目标文件新或者目标文件不存在时，才执行移动操作。

### echo 

参数为自定义内容，会将内容写到标准输出。

通常用于打印环境变量。

### cat

参数为目标文件，会将目标文件内容写到标准输出。

cat后可以跟多个文件, 后面的文件会在前面的文件的下面, 因此可以用于**按行合并**.

### paste

如果参数是两个文件，则把两文件中的相同行号整合到一行，以此构成一个文件来输出。用于**按列合并**.

- -d 或 --delimiters: 指定分隔符。
- -s 或 --serial: 更改行为变成串列进行，即每个文件所有行通过分隔符化为一行，每个文件占一行。

```bash
file1:
1 shanghai
2 beijing
3 hangzhou

file2:
SH
BJ
HZ

paste file1 file2
结果：
1 shanghai SH
2 beijing BJ
3 hangzhou HZ

paste -d: file1 file2
结果：
1 shanghai:SH
2 beijing:BJ
3 hangzhou:HZ

paste -d: -s file1 file2
结果：
1 shanghai :2 beijing:3 hangzhou 
SH:BJ:HZ
```

### tee 

参数为目标文件，会将输入（一般被管道赋予）写到目标。与重定向的区别是, 它可以sudo.

`echo 3 | sudo tee brightness`将3写入到brightness文件。

### xargs

参数为要执行的命令，可以将<u>输入</u>作为那个命令的**参数**去执行。管道运算符`|`仅仅是把数据放入标准输入, 但命令可能不会读取标准输入, 例如`cat`在无参数时会读取标准输入, 但一旦有参数就会无视标准输入, 这时就需要使用`xargs`.

`ls | xargs rm`会删除当前目录中的所有文件。

### grep

按行搜索，输出所有匹配行。

```shell
grep [options] pattern [files]
```

- `-i`：忽略大小写进行匹配(ignore)。
- `-v`：反向查找，只打印不匹配的行(invert)。
- `-n`：显示匹配行的行号。
- `-r`：递归查找子目录中的文件。
- `-l`：只打印匹配的文件名。
- `-c`：只打印匹配的行数。

### find

```shell
find [路径] [匹配条件] [动作]
```

**路径** 是要查找的目录路径，可以是一个目录或文件名，也可以是多个路径，多个路径之间用空格分隔，如果未指定路径，则默认为当前目录。

**expression** 是可选参数，用于指定查找的条件，可以是文件名、文件类型、文件大小等等。

匹配条件 中可使用的选项有二三十个之多，以下列出最常用的部份：

- `-name pattern`：按文件名查找，支持使用通配符 `*` 和 `?`。
- `-type type`：按文件类型查找，可以是 `f`（普通文件）、`d`（目录）、`l`（符号链接）等。
- `-size [+-]size[cwbkMG]`：按文件大小查找，支持使用 `+` 或 `-` 表示大于或小于指定大小，单位可以是 `c`（字节）、`w`（字数）、`b`（块数）、`k`（KB）、`M`（MB）或 `G`（GB）。
- `-mtime days`：按修改时间查找，支持使用 `+` 或 `-` 表示在指定天数前或后，days 是一个整数表示天数。
- `-user username`：按文件所有者查找。
- `-group groupname`：按文件所属组查找。

**动作:** 可选的，用于对匹配到的文件执行操作，比如删除、复制等。

find 命令中用于时间的参数如下：

- `-amin n`：查找在 n 分钟内被访问过的文件。
- `-atime n`：查找在 n\*24 小时内被访问过的文件。
- `-cmin n`：查找在 n 分钟内状态发生变化的文件（例如权限）。
- `-ctime n`：查找在 n\*24 小时内状态发生变化的文件（例如权限）。
- `-mmin n`：查找在 n 分钟内被修改过的文件。
- `-mtime n`：查找在 n\*24 小时内被修改过的文件。

在这些参数中，n 可以是一个正数、负数或零。正数表示在指定的时间内修改或访问过的文件，负数表示在指定的时间之前修改或访问过的文件，零表示在当前时间点上修改或访问过的文件。

例: 在 src/ 目录及其子目录中查找所有扩展名为 .c 或 .S 的文件: `find src/ -name "*.[cS]"`.

联合grep: `find / -type f -name "*.log" | xargs grep "ERROR"`

### less

可以分页地查看输入或参数指定文件。**more**是其简化版本.

### sort

对输入的数据(以行为单位)排序（默认升序）后输出。

- -u: <u>去除重复行</u>。
- -r: reverse，降序。
- -o: 指定输出文件。输出重定向无法做到输出到原文件，此时需要-o。
- -t: 指定分隔符，以此划分出列。
- -k: 指定排序用的哪一列。当对不同的列进行不同的排序的时候，需要用到k来指定起始列和终止列，如`-k2,2n`表示仅对第二列（2开始2结束）进行数值比较。
- -f: 忽略大小写。
- -n: 以数字排序（而非以字符排序）。
- -b: 忽略前导空格。

### uniq

删除参数指定的输入中连续重复出现的行，只留下一行，即对连续的相同的行进行压缩去重。

配合sort，即可将全文无论何处重复的行都归一。

- -c: 在每列旁边显示该行重复出现的次数。
- -d: 仅显示重复出现的行列，显示一行。  
- -D: 显示所有重复出现的行列，有几行显示几行。  
- -u: 仅显示出一次的行列  
- -i: 忽略大小写字符的不同
- -f Fields: 忽略比较指定的列数。
- -s N: 忽略比较前面的N个字符。
- -w N: 对每行第N个字符以后的内容不作比较。

### head & tail

- head 输出文件的头部
- tail 输出文件的尾部

- -n: 指定输出行数。
- -c: 指定输出字符数。

### bc

一个计算器，将输入作为算式来计算输出。

配合`paste -sd+`即可快速生成所有行的求和式，再传入bc进行求和：

```bash
xxx | paste -sd+ | bc -l
```

### du

disk usage, 用于查询目录或文件的大小.

```shell
# du log2012.log 
300     log2012.log
```

```shell
# du -h test
608K    test/test6
308K    test/test4
4.0K    test/scf/lib
4.0K    test/scf/service/deploy/product
4.0K    test/scf/service/deploy/info
12K     test/scf/service/deploy
16K     test/scf/service
4.0K    test/scf/doc
4.0K    test/scf/bin
32K     test/scf
8.0K    test/test3
1.3M    test
```

## 系统管理

### top

实时系统监控工具.

- `-d <秒数>`：指定 top 命令的刷新时间间隔，单位为秒。
- `-n <次数>`：指定 top 命令运行的次数后自动退出。
- `-p <进程ID>`：仅显示指定进程ID的信息。
- `-u <用户名>`：仅显示指定用户名的进程信息。
- `-H`：在进程信息中显示线程详细信息。
- `-i`：不显示闲置（idle）或无用的进程。
- `-b`：以批处理（batch）模式运行，直接将结果输出到文件。
- `-c`：显示完整的命令行而不截断。
- `-S`：累计显示进程的 CPU 使用时间。

显示内容:
- 总体系统信息：
	- uptime：系统的运行时间和平均负载。
	- tasks：当前运行的进程和线程数目。
	- CPU：总体 CPU 使用率和各个核心的使用情况。
		- %us：表示用户空间程序的cpu使用率（没有通过nice调度）
		- %sy：表示系统空间的cpu使用率，主要是内核程序。
		- %ni：表示用户空间且通过nice调度过的程序的cpu使用率。
		- %id：空闲cpu
		- %wa：cpu运行时在等待io的时间
		- %hi：cpu处理硬中断的数量
		- %si：cpu处理软中断的数量
		- %st：被虚拟机偷走的cpu
	- 内存（Memory）：总体内存使用情况、可用内存和缓存。
- 进程信息：
	- PID：进程的标识符。
	- USER：运行进程的用户名。
	- PR（优先级）：进程的优先级。
	- NI（Nice值）：进程的优先级调整值。
	- VIRT（虚拟内存）：进程使用的虚拟内存大小。
	- RES（常驻内存）：进程实际使用的物理内存大小。
	- SHR（共享内存）：进程共享的内存大小。
	- %CPU：进程占用 CPU 的使用率。
	- %MEM：进程占用内存的使用率。
	- TIME+：进程的累计 CPU 时间。
- 功能和交互操作：
	- 按键命令：在 top 运行时可以使用一些按键命令进行操作，如按下 "k" 可以终止一个进程，按下 "h" 可以显示帮助信息等。
	- 排序：可以按照 CPU 使用率、内存使用率、进程 ID 等对进程进行排序。
	- 刷新频率：可以设置 top 的刷新频率，以便动态查看系统信息。

### ps

查看进程信息.

例: 查看php相关进程:
```shell
# ps -ef | grep php
root       794     1  0  2020 ?        00:00:52 php-fpm: master process (/etc/php/7.3/fpm/php-fpm.conf)
www-data   951   794  0  2020 ?        00:24:15 php-fpm: pool www
www-data   953   794  0  2020 ?        00:24:14 php-fpm: pool www
www-data   954   794  0  2020 ?        00:24:29 php-fpm: pool www
...
```


### kill

给进程发信号. 默认发送`SIGTERM`(15)信号.

使用 `kill -l` 命令列出所有可用信号。最常用的信号是：
- `SIGKILL`（信号9）：立即结束进程，不能被捕获或忽略。
- `SIGTERM`（信号15）：正常结束进程，可以被捕获或忽略。
- `SIGSTOP`（信号19）：暂停进程，不能被捕获、忽略或结束。
- `SIGCONT`（信号18）：继续执行被暂停的进程。
- `SIGINT`（信号2）：通常是Ctrl+C产生的信号，可以被进程捕获或忽略。

使用`-9`就能发送`SIGKILL`(9)信号, 或者`-s SIGKILL`.

例: 强杀进程1234: `kill -9 1234`

## cheat sheet

查询所有被include过的头文件:
```bash
find . -name "*.[ch]" | xargs grep "#include" | sort | uniq
```

递归统计指定文件名的所有文件的行数，还能排除路径中包含指定字符的文件:
```shell
find . "(" -name "*.py" -or -name "*.m" ")" -print | grep -v "node_modules" | xargs wc -l
```

# 工具

## tmux

tmux是终端复用器，使得窗口与会话分离。

[Tmux 使用教程](http://www.ruanyifeng.com/blog/2019/10/tmux.html)

## ccache

`make clean`重新编译、且源文件不变时，ccache能把之前编译的结果给直接挪过来。

# 杂项

## 日志

对于 UNIX 系统来说，程序的日志通常存放在 `/var/log`。例如， [NGINX](https://www.nginx.com/) web 服务器就将其日志存放于`/var/log/nginx`。

目前，系统开始使用 **system log**，您所有的日志都会保存在这里。大多数（但不是全部的）Linux 系统都会使用 `systemd`，这是一个系统守护进程，它会控制您系统中的很多东西，例如哪些服务应该启动并运行。`systemd` 会将日志以某种特殊格式存放于`/var/log/journal`，您可以使用 [`journalctl`](http://man7.org/linux/man-pages/man1/journalctl.1.html) 命令显示这些消息。

类似地，在 macOS 系统中是 `/var/log/system.log`，但是有更多的工具会使用系统日志，它的内容可以使用 [`log show`](https://www.manpagez.com/man/1/log/) 显示。

对于大多数的 UNIX 系统，您也可以使用[`dmesg`](http://man7.org/linux/man-pages/man1/dmesg.1.html) 命令来读取内核的日志。

如果您希望将日志加入到系统日志中，您可以使用 [`logger`](http://man7.org/linux/man-pages/man1/logger.1.html) 这个 shell 程序。下面这个例子显示了如何使用 `logger`并且如何找到能够将其存入系统日志的条目。

不仅如此，大多数的编程语言都支持向系统日志中写日志。

```
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

正如我们在数据整理那节课上看到的那样，日志的内容可以非常的多，我们需要对其进行处理和过滤才能得到我们想要的信息。

如果您发现您需要对 `journalctl` 和 `log show` 的结果进行大量的过滤，那么此时可以考虑使用它们自带的选项对其结果先过滤一遍再输出。还有一些像 [`lnav`](http://lnav.org/) 这样的工具，它为日志文件提供了更好的展现和浏览方式。
## 配置文件

一些工具也可以通过**点文件**进行配置：

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` 和 `~/.vim` 目录
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

我们应该如何管理这些配置文件呢，它们应该在它们的文件夹下，并使用版本控制系统进行管理，然后通过脚本将其 **符号链接** 到需要的地方。这么做有如下好处：

- **安装简单**: 如果您登录了一台新的设备，在这台设备上应用您的配置只需要几分钟的时间；
- **可移植性**: 您的工具在任何地方都以相同的配置工作
- **同步**: 在一处更新配置文件，可以同步到其他所有地方
- **变更追踪**: 您可能要在整个程序员生涯中持续维护这些配置文件，而对于长期项目而言，版本历史是非常重要的

>配置文件中需要放些什么？您可以通过在线文档和[帮助手册](https://en.wikipedia.org/wiki/Man_page)了解所使用工具的设置项。另一个方法是在网上搜索有关特定程序的文章，作者们在文章中会分享他们的配置。还有一种方法就是直接浏览其他人的配置文件：您可以在这里找到无数的[dotfiles 仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) —— 其中最受欢迎的那些可以在[这里](https://github.com/mathiasbynens/dotfiles)找到（我们建议您不要直接复制别人的配置）。[这里](https://dotfiles.github.io/) 也有一些非常有用的资源。missing semester课程的老师们也在 GitHub 上开源了他们的配置文件： [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles)。

### 增加可移植性

使用if语句判断环境：

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# 使用和 shell 相关的配置时先检查当前 shell 类型
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# 您也可以针对特定的设备进行配置
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

如果支持include的话，可以用它导入信息。如`~/.gitconfig`可以这样写：

```gitconfig
[include]
    path = ~/.gitconfig_local
```

然后我们可以在日常使用的设备上创建配置文件`~/.gitconfig_local`来包含与该设备相关的特定配置。甚至应该创建一个单独的代码仓库来管理这些与设备相关的配置。

这种include形式的逻辑还能用于共享配置文件。如想在 `bash` 和 `zsh` 中同时启用一些别名，可以把它们写在`.aliases`里，然后在这两个shell里应用：

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

## `source script.sh` 和 `./script.sh` 的区别

这两种情况 `script.sh` 都会在bash会话中被读取和执行，不同点在于哪个会话执行这个命令。 对于 `source` 命令来说，命令是在当前的bash会话中执行的，因此当 `source` 执行完毕，对当前环境的任何更改（例如更改目录或是定义函数）都会留存在当前会话中。 单独运行 `./script.sh` 时，当前的bash会话将启动新的bash会话（实例），并在新实例中运行命令 `script.sh`。 因此，如果 `script.sh` 更改目录，新的bash会话（实例）会更改目录，但是一旦退出并将控制权返回给父bash会话，父会话仍然留在先前的位置（不会有目录的更改）。 同样，如果 `script.sh` 定义了要在终端中访问的函数，需要用 `source` 命令在当前bash会话中定义这个函数。否则，如果你运行 `./script.sh`，只有新的bash会话（进程）才能执行定义的函数，而当前的shell不能。
## 开关端口
CentOS默认关闭端口，需要手动打开。

```bash
#查看已开放的端口
firewall-cmd --list-ports

#开放端口（开放后需要要重启防火墙才生效）
firewall-cmd --zone=public --add-port=3338/tcp --permanent

#重启防火墙
firewall-cmd --reload

#关闭端口（关闭后需要要重启防火墙才生效）
firewall-cmd --zone=public --remove-port=3338/tcp --permanent

#关闭防火墙且重启也不会重新启动
systemctl disable firewalld.service
```

但Ubuntu22.04似乎不能用firewall-cmd，可能需要用ufw。[Ubuntu 22.04 LTS - 开放端口，删除已经开放的端口\_ufw删除端口-CSDN博客](https://blog.csdn.net/qq_29761395/article/details/124791460)
## 开启路由
路由开启后，当前系统就拥有了路由转发功能，能让一个网卡收集到的信息传递给其他网卡，可以实现如Docker等对外部的连接。
```bash
vim /etc/sysctl.conf

#配置转发
net.ipv4.ip_forward=1

#重启服务，让配置生效
systemctl restart network

#查看是否成功,如果返回为“net.ipv4.ip_forward = 1”则表示成功
sysctl net.ipv4.ip_forward
```

## 配置网络
```bash
#查看网络配置
ip addr
```

```bash
#配置IP地址网关
cd /etc/sysconfig/network-scripts/ 
 
vim ifcfg-ens33  //注意：用ip addr查看，编辑对应的ensxx
 
#修改以下内容
BOOTPROTO=static  //启用静态IP地址
ONBOOT=yes      //开启自动启用网络连接

#虚拟机使用net的话，这三个值要参考vmware的vmnet8相关配置
IPADDR=192.168.126.124    //设置IP地址
NETMASK=255.255.255.0   //子网掩码
GATEWAY=192.168.126.2   //设置网关
```

```bash
#配置DNS
vim /etc/resolv.conf

nameserver 114.114.114.114
```

## 环境变量
每次用`source`配置环境变量之后，重新开机都会消失。那么，只需要在`~/.bashrc`里面写入`source`语句，则每次开始会话时都会调用该语句，也就自动配置好环境变量了。

## 守护进程

Linux 中的 `systemd`（the system daemon）是最常用的配置和运行守护进程的方法。运行 `systemctl status` 命令可以看到正在运行的所有守护进程。这里面有很多可能你没有见过，但是掌管了系统的核心部分的进程：管理网络、DNS解析、显示系统的图形界面等等。用户使用 `systemctl` 命令和 `systemd` 交互来`enable`（启用）、`disable`（禁用）、`start`（启动）、`stop`（停止）、`restart`（重启）、或者`status`（检查）配置好的守护进程及系统服务。

`systemd` 提供了一个很方便的界面用于配置和启用新的守护进程或系统服务。下面的配置文件使用了守护进程来运行一个简单的 Python 程序。文件的内容非常直接所以我们不对它详细阐述。`systemd` 配置文件的详细指南可参见 [freedesktop.org](https://www.freedesktop.org/software/systemd/man/systemd.service.html)。

```python
# /etc/systemd/system/myapp.service
[Unit]
# 配置文件描述
Description=My Custom App
# 在网络服务启动后启动该进程
After=network.target

[Service]
# 运行该进程的用户
User=foo
# 运行该进程的用户组
Group=foo
# 运行该进程的根目录
WorkingDirectory=/home/foo/projects/mydaemon
# 开始该进程的命令
ExecStart=/usr/bin/local/python3.7 app.py
# 在出现错误时重启该进程
Restart=on-failure

[Install]
# 相当于Windows的开机启动。即使GUI没有启动，该进程也会加载并运行
WantedBy=multi-user.target
# 如果该进程仅需要在GUI活动时运行，这里应写作：
# WantedBy=graphical.target
# graphical.target在multi-user.target的基础上运行和GUI相关的服务
```

如果你只是想定期运行一些程序，可以直接使用 [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html)。它是一个系统内置的，用来执行定期任务的守护进程。

## FUSE（用户空间文件系统）

[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace)（用户空间文件系统）允许运行在用户空间上的程序实现文件系统调用，并将这些调用与内核接口联系起来。在实践中，这意味着用户可以在文件系统调用中实现任意功能。

FUSE 可以用于实现如：一个将所有文件系统操作都使用 SSH 转发到远程主机，由远程主机处理后返回结果到本地计算机的虚拟文件系统。这个文件系统里的文件虽然存储在远程主机，对于本地计算机上的软件而言和存储在本地别无二致。`sshfs`就是一个实现了这种功能的 FUSE 文件系统。

一些有趣的 FUSE 文件系统包括：

- [sshfs](https://github.com/libfuse/sshfs)：使用 SSH 连接在本地打开远程主机上的文件
- [rclone](https://rclone.org/commands/rclone_mount/)：将 Dropbox、Google Drive、Amazon S3、或者 Google Cloud Storage 一类的云存储服务挂载为本地文件系统
- [gocryptfs](https://nuetzlich.net/gocryptfs/)：覆盖在加密文件上的文件系统。文件以加密形式保存在磁盘里，但该文件系统挂载后用户可以直接从挂载点访问文件的明文
- [kbfs](https://keybase.io/docs/kbfs)：分布式端到端加密文件系统。在这个文件系统里有私密（private），共享（shared），以及公开（public）三种类型的文件夹
- [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html)：方便用户浏览删除重复数据后的压缩加密备份

## 查询当前CPU的线程信息

```bash
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

## 查看一个文件夹下各文件的硬盘占用

`-d1`表示只显示一层。

```bash
du -ah -d1 文件夹路径
```

# 问题

## `bash\r`相关报错

`sh`或其他被执行、扫描的文本文件出现`bash\r`相关报错，是因为该文件格式为doc。可能是从windows移过来的，所以会有`\r`符号。

可以在vim内使用`:set ff`查看文本格式，并使用`:set ff=unix`来修改格式。







