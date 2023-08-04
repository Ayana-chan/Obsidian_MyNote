

两部分组成：
- a set of **replica groups**. Each replica group is responsible for a subset of the shards. A replica consists of a handful of servers that use Raft to replicate the group's shards. 
- **shard controller**. The shard controller decides which replica group should serve each shard; this information is called the **configuration**.

Clients consult the shard controller in order to find the replica group for a key, and replica groups consult the controller in order to find out what shards to serve. There is a single shard controller for the whole system, implemented as a fault-tolerant service using Raft.

If before, the Put should take effect and the new owner of the shard will see its effect; if after, the Put won't take effect and client must re-try at the new owner.如果put请求被安排到重配置之前执行的话，则会正确执行，并且其改动能被新的掌管此shard的server得知；之后执行的话，它应该被丢掉。

重配置操作也当做log。

You will need to ensure that at most one replica group is serving requests for each shard at any one time.

Reconfiguration also requires interaction among the replica groups. For example, in configuration 10 group G1 may be responsible for shard S1. In configuration 11, group G2 may be responsible for shard S1. During the reconfiguration from 10 to 11, G1 and G2 must use RPC to move the contents of shard S1 (the key/value pairs) from G1 to G2. shard所有权转移后，内容也要通过rpc从旧的发往新的。

>You don't have to implement Raft cluster membership changes.

Lab4需要抄大量lab3造出的kvraft代码，要是出现bug可能要两面改，因此开工前三思。
# A

replica group identifiers (GID). The shardctrler should allow re-use of a GID if it's not part of the current configuration (i.e. a GID should be allowed to Join, then Leave, then Join again). GID不可重复但可复用。

## join

The shardctrler should react by creating a new configuration that includes the new replica groups.

The new configuration should divide the shards as evenly as possible among **the full set of groups**, and should move as few shards as possible to achieve that goal.

## leave

The shardctrler should create a new configuration that does not include those groups, and that assigns those groups' shards to the remaining groups. 删掉一些group并把它们拆分到各个其他group中。

The new configuration should divide the shards as evenly as possible among the groups, and should move as few shards as possible to achieve that goal.

## move

The shardctrler should create a new configuration in which the shard is assigned to the group.

不知道啥意思的一段话： The purpose of Move is to allow us to test your software. A Join or Leave following a Move will likely un-do the Move, since Join and Leave re-balance.

## query

The shardctrler replies with the configuration that has that configuration number.

**If the number is -1 or bigger than the biggest known configuration number, the shardctrler should reply with the latest configuration.**

The result of Query(-1) should reflect every Join, Leave, or Move RPC that the shardctrler finished handling before it received the Query(-1) RPC. 线性一致性

## 其他

- The very first configuration should be numbered **zero**.
- It should contain no groups.
- all shards should be assigned to GID **zero** (an invalid GID).
- The next configuration (created in response to a Join RPC) should be numbered 1
相当于0是默认位置，是无效的。

There will usually be significantly more shards than groups (i.e., each group will serve more than one shard), in order that load can be shifted at a fairly fine granularity. shard比server多，就可以让任务粒度变小，平衡起来也比较方便。

要过滤重复请求，和kvraft一样。A不会测试这个，B会。

The code in your state machine that performs the shard rebalancing needs to be deterministic. In Go, map iteration order is [not deterministic](https://blog.golang.org/maps#TOC_7.). map遍历是无序的，可能导致若干个gid被一视同仁，导致相同的输出产生不同的结果，这不deterministic

go的map是引用，想复制的话就得重新make一个然后遍历复制。op里面是servers也是map，会产生**raft的persist时的encode**与**对servers的操作**冲突；但仔细分析发现，op里map的内容在raft层是只读的，因此apply的时候kv应当对map深拷贝。

shard数量**一般**比服务器数量大，但也有小的时候！！！此时有的gid分配不到任何任务！！！


一种保证确定性的方法是，如果一个map是gid->shards，则我们可以把这个map的所有gid放进数组然后排序，之后对数组进行遍历，然后用遍历到的gid去访问map。

如果同时join多个group的话它们被分配到的shard不一定一样，如3,3,4加了仨服务器后是2,2,2,**2,1,1**

todo：是不是之前也做过有序性？




















