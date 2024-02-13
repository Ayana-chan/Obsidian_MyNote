

>we recommend using the CRDT-consensus component.

docker似乎是可以模拟网络状况的，也可以限制资源使用。

CRDT

应该可以通过在allocations里写多个不同的策略，然后在添加节点的时候指定要使用的策略。

>使用 Docker API，您可以发送 HTTP 请求到 Docker Daemon 来获取容器的资源使用情况。您可以使用任何支持 HTTP 请求的编程语言来实现这一点，比如 JavaScript、Python、Go 等。

merkle DAG中的任意一个结点都可以引出一个子DAG，代表这个节点相关的数据。若一个结点拥有一个父结点，则说明该结点所代表的数据被该父结点包含。一个结点可以有多个父结点，因为一段数据可以被多处包含（体现了merkle DAG的去重特性）。同时，由于merkle特性，每个DAG结点都可以检测其数据是否被篡改。

哈希冲突似乎是有可能发生的

断联的两部分IPFS网络连接后似乎会发生数据同步（如去重）

todo学习DHT（kademlia）  Ruslan R, Zailani A S M, Zukri N H M, et al. Routing performance of structured overlay in Distributed Hash Tables (DHT) for P2P[J]. Bulletin of Electrical Engineering and Informatics, 2019, 8(2): 389-395.dht

>The CID you will retrieve is actually a folder containing a single image file. The reason for this that when files are added to IPFS, the filename is not stored by default. To retain the filename, it's a common practice to wrap the file in a directory. In such instances, you end up with two CIDs - one for the file and another for the directory containing the file.

IPFS应该可以防止无意义数据的恶意上传。gpt4：如果一个恶意节点尝试塞入大量无意义的数据，这些数据只会影响该节点自身，因为在 IPFS 中，其他节点只会请求和存储它们需要的数据。这意味着，除非其他节点主动去请求这些无意义的数据，否则这些数据不会传播到网络的其他部分。

>When an IPFS command executes without parameters, the CLI client checks whether the `$IPFS_PATH/api` file exists and connects to the address listed there.

>If you already have an IPFS node on your computer, IPFS Desktop will act as a control panel and file browser for that node. If you don't have a node, it'll install one for you. And either way, IPFS Desktop will automatically check for updates. 考虑用desktop监视？
>**Explore the "Merkle Forest" of IPFS files** with a visualizer that lets you see firsthand how example datasets stored on IPFS — or your own IPFS files — are broken down into content-addressed pieces

>gpt3.5: 要监视局域网中的多个IPFS节点的存储方式和状态，您可以按照以下步骤操作：  
打开IPFS Desktop应用程序并确保您已连接到局域网中的IPFS节点。  
在IPFS Desktop的界面中，您可以在"Peers"或"节点"选项卡下找到连接的IPFS节点列表。  
选择您想要监视的特定节点，然后查看其存储使用情况和状态信息。  
您还可以使用IPFS Desktop提供的图形化界面来查看节点之间的连接情况和数据传输状态。  
通过这些步骤，您可以利用IPFS Desktop监视局域网中的多个IPFS节点的存储方式和状态

todo 了解一下docker swarm

todo 使用compose 的depend on

IPFS节点下载的东西太多会进行gc，把以前缓存的东西删掉，而pin住的数据不会被删。

pinning service则是一些厂商在收费后提供若干IPFS节点来提供持久化服务。

>There are different types of IPFS nodes. And depending on the use-case, a single IPFS node can serve one of many functions:

- [Relay](https://docs.ipfs.tech/concepts/nodes/#relay)
- [Bootstrap](https://docs.ipfs.tech/concepts/nodes/#bootstrap)
- [Delegate routing](https://docs.ipfs.tech/concepts/nodes/#delegate-routing-node)

原始的bootstrap只能连接单一的bootstrap节点，但新特性使得可连接的peer都会被缓存，挂一个还能连另一个。

对上传下载的用户友好的界面

Kubo可以用`localhost:5001/webui`来查看ui控制面板，但只能管理存在MFS里的文件。

`ipfs files`带头的命令才能控制MFS。

TODO：如果一个节点是无私贡献者，会不会导致其他节点还不清债？？

>In IPFS, a **block** refers to a single unit of data, identified by its key (hash). A block can be any sort of data, and does not necessarily have any sort of format associated with it. An **object**, on the other hand, refers to a block that follows the Merkle DAG protobuf data format. It can be parsed and manipulated via the `ipfs object` command. Any given hash may represent an object or a block

>When making your own block data, you won't be able to read the data with `ipfs cat`. This is because you are inputting raw data without the UnixFS data format. To read raw blocks, use `ipfs block get` as shown in the example.

到底什么是bootstrap？

也许需要一个IPFS节点作为网关（或客户端）来使用，以测试能否获取数据。

Same-origin policy: 只有当协议、网站名、端口号完全相同的网站之间才能跳转。使用`https://{CID A}.ipfs.{gatewayURL}/{webpage A}`（subdomain gateway）即可利用此机制防止跨CID（跨源）跳转。[Site Unreachable](https://en.wikipedia.org/wiki/Same-origin_policy)


```dockerfile
# 使用Kubo的官方Docker镜像作为基础镜像
FROM ipfs/kubo:latest

# 安装依赖项和监控工具
# 这里假设你的监控工具可以通过包管理器安装
RUN apk add --no-cache <你的监控工具包名>

# 或者，如果你的监控程序是一个二进制文件，你可以这样添加它：
# COPY path_to_your_monitoring_program /usr/local/bin/your_monitoring_program

# 如果你的监控程序是一个脚本，确保它具有执行权限
# COPY path_to_your_monitoring_script /usr/local/bin/your_monitoring_script
# RUN chmod +x /usr/local/bin/your_monitoring_script

# (可选) 如果你的监控程序需要配置文件，可以添加配置文件
# COPY path_to_your_config_file /path/in/container/config_file

# (可选) 添加启动脚本，如果你需要在启动容器时运行一些初始化命令
# COPY start.sh /usr/local/bin/start.sh
# RUN chmod +x /usr/local/bin/start.sh

# 暴露任何需要的端口
# EXPOSE <你的监控工具端口>

# 设置容器启动时执行的命令
# 这里假设你的监控程序可以作为后台服务运行
# CMD ["your_monitoring_program"]

# 或者，如果你有一个启动脚本来启动Kubo和你的监控程序，可以这样设置：
# CMD ["/usr/local/bin/start.sh"]
```

```yaml
version: '3'
services:
  kubo:
    build: .
    ports:
      - "4001:4001" # Libp2p
      - "5001:5001" # API
      - "8080:8080" # Gateway
    volumes:
      - ./data/ipfs:/data/ipfs
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 1G

  cadvisor:
    image: google/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8081:8080"
```

`ipfs dag get <root-hash>`可以获取merkle树（真的吗？），然后应该只能手动解析了

也许：api调用get的时候，IPFS会将文件搜索并缓存到其repo中，然后将本地流交给程序。

也许可以使用选举，选出一个节点统筹全部信息？然后各个节点互相协作以调整副本。节点之间如何通信是个大问题。加快响应速度则独立实现网关。看看cluster有没有更合理的东西。

也许可以使用桶，让IPFS节点按照性能均匀分配到各个桶中，每个监控程序管理一个桶；当监控程序数量变化的时候，就可以增删桶并进行负载均衡。

也许可以直接假设只有较少的IPFS节点，用一个监控程序管理。如果要扩展的话，就用完全另一部分监控程序和IPFS节点即可，缺点就是需要额外管理IPFS节点的归属，需要尽力保证IPFS节点的分散程度以求平衡，同时节点迁移的时候代价较大（一个问题：如果一个节点里面都是非pin的数据的话，监控程序能发现吗？）。

>IPFS cluster使用时要关闭gc，防止pin前被删。但裸IPFS会不会也有此问题？： IPFS garbage collection should be disabled while adding. Because blocks are block/put individually, if a GC operation happens while and adding operation is underway, and before the blocks have been pinned, they would be deleted.

unable to connect to agent pipe

```
docker run -d --name ipfs_host \
-v $PWD/staging:/export \
-v $PWD/data:/data/ipfs \
-p 4001:4001 -p 4001:4001/udp -p 127.0.0.1:8080:8080 -p 127.0.0.1:5001:5001 \
ipfs/kubo:latest
```

暴露到外网：
```
docker run -d --name ipfs_host \
-v $PWD/staging:/export \
-v $PWD/data:/data/ipfs \
-p 4001:4001 -p 4001:4001/udp -p 8080:8080 -p 5001:5001 \
ipfs/kubo:latest
```

看看swarm相关的api

[axum/examples/cors/src/main.rs at main · tokio-rs/axum · GitHub](https://github.com/tokio-rs/axum/blob/main/examples/cors/src/main.rs)

看看为什么api只能是static

使用Gateway来get的时候可以使用NoCache来防止IPFS去主动下载。

应该有一个网关直接把控各个IPFS的各种行为，而网站提供的服务则是另外的无状态服务器集群，集群会访问IPFS。

这样设计的话，可能不需要NoCache，让每个IPFS节点作为易失效CSN服务器使用。

但是不能暴露RPC！！

也许，写个web服务将RPC的一部分用于向公网暴露也是不错的！应该还要写个访问控制。这样也是必然造成内存复制，只不过将复制工作分散了。但cluster是不是也是允许复制呢？

cluster的API的add：[ipfs-cluster/adder/adderutils/adderutils.go at master · ipfs-cluster/ipfs-cluster · GitHub](https://github.com/ipfs-cluster/ipfs-cluster/blob/master/adder/adderutils/adderutils.go#L22)。

使用类似于CDN的架构可以帮助选择响应服务器、收集热度信息

可以让文件的上传和下载用一个无状态的网关来进行，相关数据直接存到同一个数据库里面，然后监控程序定期查看数据库以进行决策。节点的选择是个问题，但可以参考nginx。

可以做一个纯粹的pin service，单纯通过监控各节点情况来动态调整副本。热度这种东西，应该是会直接影响节点情况的。不过很难知道特定文件会带给服务器的压力大小，那就很难调整文件的副本量。但是，副本量可以交给用户来规定，规定方式可以参考cluster，毕竟买的时候就应该进行指定来衡量价钱。一个文件带来的压力较小，可能是热度不高，也有可能是外部IPFS节点缓存了很多。实际上，应该只需要保证“旗下集群收到get请求的时候保证可用性”即可。这种情况下对外开放IPFS节点带来的唯一副作用应该是“让访问量的波动因素（规律）变得更为复杂”。

但是，如果不检测节点压力的话，对外开放IPFS节点后，其他IPFS网关会给节点以额外的压力，节点可能检测不到。

由于读>>写，因此只需要注意收集每个文件的相关get请求数量，而add则完全可以交由pin service服务器一起完成。

CDN可以被绕过，那么这里用一个服务来给予访问建议让用户自觉遵守应该没什么问题。但依旧要设网关来管理访问量。

如果pin策略也做成分布式的话，要考虑到策略冲突的问题。无法容忍的情况下才需要CRDT。

每个文件都可以算出访问量，然后就能预测未来热度；于是每个节点的未来热度也就可以被预测了。可以让monitor在开始计算后再收集访问量数据，同时收集节点硬件压力。平时的pin可以使用随机算法，可容纳力越高的节点被分配的概率就越大。（可以在分配节点选完后，对节点硬盘进行查看，塞不下就重新抽）

先开发IPFS service吧！！！

有GPT4了的话再问问能不能用RPC来get

wrapper可以记录ipfs节点的延迟。

每个节点一个IPFS是否就可以互相分享配置了！！！

pin居然是同步操作，要等文件下载完毕才能返回。因此要异步地请求

TODO 修改所有&str接口！！！！！！！！！！！
