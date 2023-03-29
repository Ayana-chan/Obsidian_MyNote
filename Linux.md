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







