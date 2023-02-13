# 理论
image（镜像）相当于“类”，container（容器）即为类实现的“对象”。仓库里存的都是镜像。

![](assets/uTools_1676288415314.png)

# 使用
命令查询：[Docker 备忘清单 & docker cheatsheet & Quick Reference](http://bbs.laoleng.vip/reference/docs/docker.html)

想搜索镜像信息可以去官网：[Docker](https://hub.docker.com/)
## 命令
### 版本指定
`ImageName:3.1`可以指定使用3.1版本的ImageName（无论是下载还是运行）。
### Docker Run 参数
`-i`表示一直运行（不会因为没有客户机而停止运行），一般和`-t`一起使用。

`-t`为容器分配一个“伪输入终端”。

`-d`为后台运行（守护模式），需使用`docker exec`进入容器，在使用`exit`退出容器后不会关闭容器。

`-it`：交互式容器。`-id`：守护式容器。



























