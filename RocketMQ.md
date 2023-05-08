# 概念

- Broker: 存储、供给消息。
- NameServer: 注册Broker，通过NameServer来发现Broker。
- Producer: 生产消息。先和NameServer长连接，得知哪个Broker存有对应Topic后，就和该Broker直接连接并发送消息。
- Consumer: 消费消息。与Producer类似流程。

# 使用Docker安装

## 安装NameServer

```bash
docker run -d \
--restart=always \
--name rmqnamesrv \
-p 9876:9876 \
-v /mydata/rocketmq/logs:/root/logs \
-v /mydata/rocketmq/store:/root/store \
-e "MAX_POSSIBLE_HEAP=100000000" \
apache/rocketmq \
sh mqnamesrv 
```

## 安装Broker
先写配置文件`/mydata/broker/conf/broker.conf`：

```bash
# 所属集群名称，如果节点较多可以配置多个
brokerClusterName = DefaultCluster
#broker名称，master和slave使用相同的名称，表明他们的主从关系
brokerName = broker-a
#0表示Master，大于0表示不同的slave
brokerId = 0
#表示几点做消息删除动作，默认是凌晨4点
deleteWhen = 04
#在磁盘上保留消息的时长，单位是小时
fileReservedTime = 48
#有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
brokerRole = ASYNC_MASTER
#刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
flushDiskType = ASYNC_FLUSH
# 设置broker节点所在服务器的ip地址
brokerIP1 = 192.168.177.114
# 磁盘使用达到95%之后,生产者再写入消息会报错 CODE: 14 DESC: service not available now, maybe disk full
diskMaxUsedSpaceRatio=95
```

然后创建

```bash
docker run -d  \
--restart=always \
--name rmqbroker \
--link rmqnamesrv:namesrv \
-p 10911:10911 \
-p 10909:10909 \
-v  /mydata/broker/logs:/root/logs \
-v  /mydata/broker/store:/root/store \
-v /mydata/broker/conf/broker.conf:/home/rocketmq/rocketmq-4.9.4/conf/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=200000000" \
apache/rocketmq:4.9.4 \
sh mqbroker -c /home/rocketmq/rocketmq-4.9.4/conf/broker.conf
```
## 安装控制台

这个镜像得提前pull，否则似乎会超时：
```bash
docker pull pangliang/rocketmq-console-ng
```

```bash
docker run -d \
--restart=always \
--name rmqadmin \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.177.114:9876 \
-Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9999:8080 \
pangliang/rocketmq-console-ng
```

### 小问题
如果console的容器创建时地址写错，就会在网页右上角报：

```bash
org.apache.rocketmq.remoting.exception.RemotingConnectException: connect to failed
```














