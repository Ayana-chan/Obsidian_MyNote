# Docker基础
## 基本结构
image（镜像）相当于“类”，container（容器）即为类实现的“对象”。仓库里存的都是镜像。

想搜索镜像信息可以去官网：[Docker](https://hub.docker.com/)

![](assets/uTools_1676288415314.png)

容器**虚拟化操作系统**，虚拟机虚拟化硬件。

![](assets/uTools_1676689056862.png)
## 镜像原理

- Docker镜像是由特殊的文件系统叠加而成。
- 最底端是bootfs,并**使用宿主机的bootfs**（linux的各个发行版的bootfs基本相同，因此可直接利用。这也是为什么Docker依赖于Linux）。
- 第二层是root文件系统rootfs,称为base image。
- 然后再往上可以叠加其他的镜像文件。
- 统一文件系统(Union File System)技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角,这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统（对外暴露最顶端）。
- 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。
- 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器。

分层的目的是为了复用，一个镜像（或其组成部分）可以成为另一个镜像的依赖。

![](assets/uTools_1676522015630.png)
## 数据卷
### 概念
- 数据卷是宿主机中的一个目录或文件。
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步。
- 一个数据卷可以被多个容器同时挂载。
- 一个容器也可以挂载多个数据卷。

![](assets/uTools_1676350813096.png)
### 作用
- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换（挂载同一个数据卷）
### 数据卷操作
使用`docker inspect xx`显示xx容器的细节，在HostConfig里面的Binds可以看到数据卷关系，Mounts里也有。当然也有更专门的数据卷操作命令。

数据卷是不会自动回收的，要在删除容器时附加`-v`参数来删除相关的数据卷，或者也可以使用`docker volume prune`删除无主的数据卷。
### 数据卷容器
c3作为c1和c2的数据卷容器，会把自己的所有数据卷设置都传给c1和c2，也因此即使c3关掉了也不会影响c1和c2。

![](assets/uTools_1676352336031.png)

### 端口映射
外部机器访问宿主机，宿主机将对应端口映射到容器中，从而可以实现外部机器与容器的通信。
## 命令
命令查询：[Docker 备忘清单 & docker cheatsheet & Quick Reference](http://bbs.laoleng.vip/reference/docs/docker.html)
### 服务相关命令
使用`systemctl`对docker本身的服务进行设置：
- 启动docker服务：`systemctl start docker`
- 停止docker服务：`systemctl stop docker`
- 重启docker服务：`systemctl restart docker`
- 查看docker服务状态：`systemctl status docker`
- 设置开机启动docker服务：`systemctl enable docker`
### 版本指定
`ImageName:3.1`可以指定使用3.1版本的ImageName（无论是下载还是运行）。

实际上就是指定TAG。
### Docker Run 参数
#### 启动方式相关
`-i`表示进入交互模式，也即打开输入流。即使没有客户机也不会停止运行。一般和`-t`一起使用。

`-t`为容器分配一个“伪输入终端”。`-it`即是生成伪输入终端并开放输入流，从而可以让我们对容器进行输入。

`-d`为后台运行（守护模式），需使用`docker exec -it xxx /bin/bash`进入容器，在使用`exit`退出容器后不会关闭容器。若加上`-i`可解决连接波动的问题。

`-it`：交互式容器。`-id`：守护式容器。
#### 其他
- `-v 宿主机目录:容器内目录`：设定数据卷。必须是绝对路径。若目录不存在则会自动创建。可以缺省宿主机目录与冒号，此时会自动指定宿主机的一个文件夹（在`/var/lib/docker`里面）参与数据卷。
- `--volumes-from xx`：令xx成为当前容器的数据卷容器。
- `-p`：端口映射。宿主机帮助转发。
- `--net=host`：直接使用宿主机的对应端口（相当于运行在宿主机上）。并使`-p`失效。
### 开机自动启动
```bash
sudo docker update ContainerName --restart always
```
## 容器转镜像
```bash
#容器变成镜像
docker commit 容器id 镜像名称：版本号

#镜像生成压缩包（否则无法分享）
docker save -o 压缩文件名称 镜像名称：版本号

#压缩包生成镜像
docker load -i 压缩文件名称
```

# DockerFile
备忘查询：[Dockerfile 备忘清单 & dockerfile cheatsheet & Quick Reference](http://bbs.laoleng.vip/reference/docs/dockerfile.html)

## 部署SpringBoot项目
dockerfile写法如下：

```txt
#定义父镜像
FROM java:8

#定义作者信息
MAINTAINER ayana

#将jar包添加到容器（app.jar是容器内的jar包名）
ADD springboot_name.jar app.jar

#定义容器启动执行的命令
CMD java -jar app.jar
```

```bash
#通过dockerfile构建镜像（不加版本就是latest，写成`镜像名称 .`）
docker bulid -f dockerfile文件路径 -t 镜像名你:版本
```

# Docker Compose
Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。即使对于单个镜像，也能当成配置文件使用。使用步骤：
1. 利用Dockerfile定义运行环境镜像
2. 使用docker-compose,yml定义组成应用的各服务
3. 运行docker-compose up启动应用

`docker-compose.yml`:

```yml
version: '3'
services:
 nginx:
  image: nginx
  ports:
   - 80:80
  links:
   - app
  volumes:
   - ./nginx/conf.d:/etc/nginx/conf.d
 app:
  image: springboot_hello
  expose:
   - "8080"
```

在yml所在目录输入如下命令即可管理容器：

```bash
#启动
docker-compose -f standalone-derby.yaml up
#关闭
docker-compose -f standalone-derby.yaml stop
#移除
docker-compose -f standalone-derby.yaml rm
#关闭并移除
docker-compose -f standalone-derby.yaml down
```

开机自启动：
```bash
restart: always
```

# 具体部署
## 问题
数据卷的宿主机路径都使用了`$PSW`，因此创建容器前一定要注意终端当前目录是什么！

ping得通但连接不到时首先考虑端口开没开：

![Linux](Linux.md#开关端口)

也可能是路由没开放导致Docker无法接触外网：

![Linux](Linux.md#开启路由)
## MySQL
### 创建
以当前目录为数据卷创建mysql：

```bash
docker run -id \
--name mysql \
-p 3306:3306 \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/1ib/mysql \
-e MYSQL_ROOT_PASSWORD=1234 \
mysql:8.0.30
```

### 问题
一些权限设置：

```sql
#给予所有主机通过root访问数据库的权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

#刷新使设置生效
FLUSH PRIVILEGES;

#查表可以看到权限变化（也可以通过改表完成权限设置）
SELECT user,host FROM mysql.user;
```

外部机连接时可能出现`Public Key Retrieval is not allowed`错误，可这样解决：[MySQL 8.0的Public Key Retrival错误，毫无规律可言怎么破？ - 知乎](https://zhuanlan.zhihu.com/p/371161553)

## Tomcat
以当前目录为webapp创建Tomcat:

```bash
docker run -id \
--name tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat
```

这样只要给当前目录里面放入web项目即可被访问。

## Nginx
提前准备好配置文件`$PWD/conf/nginx.conf`：

```conf
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid /var/run/ndinx.pid;

events{
	worker_connections 1024;
}

http{
	include /etc/nginx/mime.types;
	default_type application/octet-stream;
	
	log_format main '$remote_addr - $remote_user [$time_local] "$request"'
					'$status $body_bytes_sent "$http_referer"'
					'"$http_user_agent" "$http_x_forwarded_for"';
					
	access_log /var/log/nginx/access.log main;

	sendfile on;
	#tcp_nopush on;
	
	keepalive_timeout 65;
	
	#gzip on;
	
	include /etc/nginx/conf.d/*.conf;
}
```

创建容器：

```bash
docker run -id \
--name nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx:1.22
```

## Redis
`redis-server /etc/redis/redis.conf`就是以配置文件创建容器。

```bash
docker run -id \
--name redis \
-p 6379:6379 \
-v $PWD/data:/data \
-v $PWD/conf/redis.conf:/etc/redis/redis.conf \
redis:7.0 \
redis-server /etc/redis/redis.conf
```

## Nacos
准备配置文件`$PWD/init.d/custom.properties`（未知作用）：

```properties
management.endpoints.web.exposure.include=*
```

从容器中的`/conf`中复制`application.properties`并放到`$PWD/conf/application.properties`，然后修改以设置数据库，从而可以持久化数据：

```properties
spring.datasource.platform=mysql
spring.sql.init.platform=mysql
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
db.num=${MYSQL_DATABASE_NUM:1}
db.url.0=jdbc:mysql://192.168.177.114:3306/nacos_config?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}
db.user.0=root
db.password.0=1234
```

从容器中的`/conf`复制sql，创建`nacos_config`数据库。

创建容器然后创建容器。
- 必须要开三个端口映射或者用`--net=host`否则微服务无法正常访问。
- 只有使用了`--net=host`才能在`application.properties`里将sql的url设为localhost。

```bash
docker run -id \
--name nacos \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v $PWD/init.d/custom.properties:/home/nacos/init.d/custom.properties \
-v $PWD/logs:/home/nacos/logs \
-v $PWD/conf/application.properties:/home/nacos/conf/application.properties \
--privileged=true \
--restart always \
nacos/nacos-server
```



























