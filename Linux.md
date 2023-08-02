
# 基础知识

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

# 问题

## `bash\r`相关报错

`sh`或其他被执行、扫描的文本文件出现`bash\r`相关报错，是因为该文件格式为doc。可能是从windows移过来的，所以会有`\r`符号。

可以在vim内使用`:set ff`查看文本格式，并使用`:set ff=unix`来修改格式。







