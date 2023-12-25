

>we recommend using the CRDT-consensus component.

docker似乎是可以模拟网络状况的，也可以限制资源使用。

CRDT

应该可以通过在allocations里写多个不同的策略，然后在添加节点的时候指定要使用的策略。

>使用 Docker API，您可以发送 HTTP 请求到 Docker Daemon 来获取容器的资源使用情况。您可以使用任何支持 HTTP 请求的编程语言来实现这一点，比如 JavaScript、Python、Go 等。

merkle DAG中的任意一个结点都可以引出一个子DAG，代表这个节点相关的数据。若一个结点拥有一个父结点，则说明该结点所代表的数据被该父结点包含。一个结点可以有多个父结点，因为一段数据可以被多处包含（体现了merkle DAG的去重特性）。同时，由于merkle特性，每个DAG结点都可以检测其数据是否被篡改。

哈希冲突似乎是有可能发生的

断联的两部分IPFS网络连接后似乎会发生数据同步（如去重）

todo确认：file的结构就是实际的Merkle DAG结构。

todo学习DHT Ruslan R, Zailani A S M, Zukri N H M, et al. Routing performance of structured overlay in Distributed Hash Tables (DHT) for P2P[J]. Bulletin of Electrical Engineering and Informatics, 2019, 8(2): 389-395.dht

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

todo：kademlia
















