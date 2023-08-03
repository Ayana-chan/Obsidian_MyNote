
# 基础知识

## 主要路径意义

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
## 权限

- r: read查看
- w: write修改
- x: execute执行

一般以`rwx`序列来表示权限。`-`表示不具备某权限，如`r-x`表示无权执行。


# 指令

### ls

列出当前目录下的所有文件。

- 更具体的列表: `-l` （等价于`ll`命令）
- 所有文件（包括隐藏文件）: `-a`
- 文件打印以人类可以理解的格式输出 (例如，使用454M 而不是 454279954): `-h`
- 文件以最近访问顺序排序: `-t`
- 以彩色文本显示输出结果: `--color=auto`  

### mkdir

创建目录。

### touch

创建文件。

`touch {1..10}.html`会创建`1.html`~`10.html`十个文件。

### cat

参数为目标文件，会将目标文件内容写到标准输出。

### echo 

参数为自定义内容，会将内容写到标准输出。

### tee 

参数为目标，会将输入（一般被管道赋予）写到目标。

`echo 3 | sudo tee brightness`将3写入到brightness文件。

### xargs

参数为要执行的命令，可以将输入作为那个命令的参数去执行。

`ls | xargs rm`会删除当前目录中的所有文件。

### less

可以分页地查看输入或参数指定文件。

### sort

对输入的数据排序（默认升序）后输出。

- -u: 去除重复行。
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
- -s N: 忽略比较前面的N个字符。-w N: 对每行第N个字符以后的内容不作比较。

### head & tail

- head 输出文件的头部
- tail 输出文件的尾部

- -n: 指定输出行数。
- -c: 指定输出字符数。

### paste

如果参数是两个文件，则把两文件中的相同行号整合到一行，以此构成一个文件来输出。

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

### bc

一个计算器，将输入作为算式来计算输出。

配合`paste -sd+`即可快速生成所有行的求和式，再传入bc进行求和：

```bash
xxx | paste -sd+ | bc -l
```

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




# 问题

## `bash\r`相关报错

`sh`或其他被执行、扫描的文本文件出现`bash\r`相关报错，是因为该文件格式为doc。可能是从windows移过来的，所以会有`\r`符号。

可以在vim内使用`:set ff`查看文本格式，并使用`:set ff=unix`来修改格式。







