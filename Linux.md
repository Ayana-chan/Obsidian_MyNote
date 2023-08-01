
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
# 杂项
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







