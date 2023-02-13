# 理论
image（镜像）相当于“类”，container（容器）即为类实现的“对象”。仓库里存的都是镜像。

![](assets/uTools_1676288415314.png)

# 使用
命令查询：[Docker 备忘清单 & docker cheatsheet & Quick Reference](http://bbs.laoleng.vip/reference/docs/docker.html)

想搜索镜像信息可以去官网：[Docker](https://hub.docker.com/)
## 命令
### 服务相关命令
使用`systemctl`对docker本身的服务进行设置：
- 启动docker服务：`systemctl start docker`
- 停止docker服务：`systemctl stop docker`
- 重启docker服务：`systemctl restart docker`
- 查看docker服务状态：`systemctl status docker`
- 设置开机启动docker服务：`systemctl enable docker`
### 版本指定
`ImageName:3.1`可以指定使用3.1版本的ImageName（无论是下载还是运行）。
### Docker Run 参数
`-i`表示进入交互模式，也即打开输入流。即使没有客户机也不会停止运行。一般和`-t`一起使用。

`-t`为容器分配一个“伪输入终端”。`-it`即是生成伪输入终端并开放输入流，从而可以让我们对容器进行输入。

`-d`为后台运行（守护模式），需使用`docker exec`进入容器，在使用`exit`退出容器后不会关闭容器。若加上`-i`可解决连接波动的问题。

`-it`：交互式容器。`-id`：守护式容器。



























