

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










