

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

move会破坏平衡，但会被join或leave给平衡回来。
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

go的map是引用，想复制的话就得重新make一个然后遍历复制。op里面是servers也是map，会产生**raft的persist时的encode**与**对servers的操作**冲突；但仔细分析发现，op里map的内容在raft层是只读的，因此apply的时候kv应当对map深拷贝。这个bug在lab4B的Hint里面提到了： If you put a map or a slice in a Raft log entry, and your key/value server subsequently sees the entry on the applyCh and saves a reference to the map/slice in your key/value server's state, you may have a race. Make a copy of the map/slice, and store the copy in your key/value server's state. The race is between your key/value server modifying the map/slice and Raft reading it while persisting its log.

shard数量**一般**比服务器数量大，但也有小的时候！！！此时有的gid分配不到任何任务！！！

一种保证确定性的方法是，如果一个map是gid->shards，则我们可以把这个map的所有gid放进数组然后排序，之后对数组进行遍历，然后用遍历到的gid去访问map。（似乎最后没用到）

如果同时join多个group的话它们被分配到的shard不一定一样，如3,3,4加了仨服务器后是2,2,2,**2,1,1**

能直接算出平衡后的group的shard数量。

最后的解决方法是，把所有gid按<u>主shard量</u>、<u>次gid号大小</u>来排序，这样就能保证确定性的同时，让其按shard数从大到小排序，大的把多余的shard拿出来放到池子的，小的从池子里面拿。

# B

>migration 迁移

config每次收到就去start一下。但不能只start一次，因为有概率所以服务器都在选举、不是leader，直接无效；或者一个leader受理了它、然后又因为各种原因弄丢了。但这样一直查询可能会产生很大的压力！

start之间理论上是可以互相插队的，因此config在日志中可能是乱序的。

为什么会收到比当前的config更老的config？可能是leader查到了新的config并更新了Follower的config，但Follower仍可能因为网络延迟收到过期的config。

---

似乎可以保证一个分片只能归属于一个group，且不会有log丢失问题。不会出现online变成pulling或者offline变成pushing。--但是，如果一个服务器重启，其Snapshot中可能又把shard读了回来！因此，config的num也要像raft的term一样管理。

>（已否决）只有发送方是主动的，接收方来者不拒。也许可以让接收方根本不等待？但这样的话有别人要一个shard自己又没有该怎么办？有可能一辈子都拿不到这个shard！

如果把shard的内容都放进*添加shard*的log里面，则重启也没事了！

如果有另一个group要给我发shard，我宕机了，那么那个group就会一直等待；正常工作后，则会接收shard然后将其放入raft日志保证其不会丢失，之后将回复“pull完成”。因此，若一个group宕机，所有与之相关的group都会暂停更新config。一次迁移必然是在同一个config下的，如果发现pushing的目标的config.Num偏大，则说明自己重启过，已经给对面成功推送过了（日志里面遇到gc的时候似乎也能直接取消pushing）；pulling虽然无法主动查询目标的config.Num，但pulling成功后本地会有日志，apply时识别到即可认为pulling成功。

==已完成：==重启的时候，可能会迅速的到来多个config！！！此过程中还会不断获取config，放在很靠后的日志末尾。可能要在读到新config但没完成之前的config的时候阻塞applyCh？不！由于config配置过程中会加log来标志进度，因此读到下一个config之前必然依靠log完成了当前config。

这样又会引出一个问题：由于对config是否完成的检测独立于日志的apply，因此在上一个config被标记为完成前，下一个config可能就从日志中到来了。此时直接认为上一个config完成了，标记为offline或者online。**此时要做的唯一一件实事，就是进行gc**！！！

==已完成：==每次都去要下一个config，要不到的话再开始每100ms查一次。**一个接着一个按序处理reconfig** ，处理完一个后再查新config。

只有leader在获取config塞入日志，然后所有服务器都会更新config、更新state并尝试继续处理。

发送方发现新config时，会让对应的key停止服务，并开始转移。因此config何时发现都是可以的。转移完毕后，接收方宣布开始接收此shard。转移过程中，shard是不会被读写的。

一个shard被归为gid 0时就不需要做迁移了。空group进行join的时候，没有任何服务器会发送迁移，因此要查看上一次config是否为全0。刚好初始态下num为0必然对应全0。

>If a test fails, check for gob errors (e.g. "gob: type not registered for interface ..."). Go doesn't consider gob errors to be fatal, although they are fatal for the lab.

==已完成：==config被apply的时候，当前config被更新了，但是不能立马接收新shard的服务，中间有一个pull状态。此时也回复ErrWrongGroup似乎没有副作用。

Think about how the shardkv client and server should deal with ErrWrongGroup. Should the client change the sequence number if it receives ErrWrongGroup? Should the server update the client state if it returns ErrWrongGroup when executing a Get/Put request? 如果config的log之后有大量需要被转移的shard，则这些shard也要被ErrWrongGroup。因此，如果客户端发现ErrWrongGroup的话，就需要更新一下序列号，否则server可能以为重试的请求是重复的请求。或者client不用加序列号，只需要让server在apply时发现ErrWrongGroup的话就不更新已完成序列号，这样似乎更加简洁明了。<u>上述思路是满足TestChallenge2Unaffected的。</u>

After a server has moved to a new configuration, it is acceptable for it to continue to store shards that it no longer owns (though this would be regrettable in a real system). This may help simplify your server implementation. 一个shard接收迁移的时候就把原shard都给delete了再重新给上。

You can send an entire map in an RPC request or reply, which may help keep the code for shard transfer simple. 感觉这样的话database还是保持一整块比较好？但这样的话就要先加锁、深拷贝了才能发送rpc。不过每次config变化后拷贝一次数据库就行了，因为实际要发送的shard现在都不会变化。

>During a configuration change, a pair of groups may need to move shards in both directions between them. If you see deadlock, this is a possible source.

对于发送方，迁移过程中如果config日志被Snapshot了，那么服务器宕机重启后，就不会再触发迁移。因此发送方的迁移任务要有持久化的东西保存（state）。

Challenge1：成功迁移后，将对应的shard设为gcing状态，再对其进行删除，删完后状态设为offline。并且不断检测处于gcing的shard，对其进行删除。这样，重启之后也能及时删除不属于自己的shard，并且不会在迁移完成之前删掉shard。

综上，或许不能直接用config里面的有无来判断是否接收一个请求，应当给每个shard赋予一个状态机：
- 被删除的shard，从Online变成Pushing，迁移完成后变成GCing，删除完成后变成Offline。
- 被添加的shard，从Offline变成Pulling，接收完成后变成Installing，安装完成后变成Online。（后来设计为接收时必须要求安装完成，因此跳过install状态）。
所有的shard都变成online或者offline后，config才算完成。

对于Follower来说，会直接从push变成offline，因为它的gc是被leader直接指示的。

TestChallenge2Partial也因此自然得到满足。

由于只有leader可以进行迁移，因此状态机的变化可能只有leader可以发现，因此状态转移需要用raft来同步。由于leader是可换的，因此状态的同步非常重要。

---

新的两种Op：
- 用于安装shard，正在pull的shard收到迁移时添加，apply后安装shard。
- 用于删除shard，push的shard完成迁移后添加，apply后删除正在gc态的shard。

只有两种方式能确保start成功：
1. 连续不断地start直到发生标志性变化
2. 等待apply

如果要让push成功之后不再重复发送，就不能保证发送完后百分百完成gc。或许有另一种消耗比较大的解决方法，让push可以重复发送，在gc原子操作完毕后停止。（这种方法消耗真的大吗？好像并没有大多少。只是要注意别发出gc后的数据，不然对方服务器重启后尚未接收到正常的installShard的log，就提前收到了非法的MigrateShard。另外，如果对方服务器成功后宕机，可能导致本地重启后无法gc，有点不合理）

实际操作database时要极为谨慎，如果config和state中途变换，可能让正在操作中的database开始接收服务，导致race。

- 如果一个group宕机很久，其启动的时候收到很卡的Client请求，发现自己的config允许接收，则顺利操作成功；然后收到了新的（对其他group来说很老的）config，并试图把这块shard push给别人，别人说你太老了，于是直接判定成功，那个请求也就丢失了。可以在client请求中添加configNum判定，**当服务器发现自己的config比client新时再继续受理**。这也不会影响效率或者不相关性，因为本来就要起码等对应的config跟上后才算合法的请求处理。
- **即使服务器的configNum偏大也不能受理**，无法保证不被删。
- 综上：只有configNum严格等同时，client请求才会被受理。这样拥有完全合理的一致性，如同term。

请求时对config相关的检查也要在apply时重做一遍，因为谁也不知道start之前发生了什么事。

由于client会不断更新configNum，因此如果client在上个config进行了一次操作，在下一个config还能再进行操作。而由于对所有group来说，requestId都是单调递增的，因此迁移的时候直接发送requestId、取大覆盖是没有任何问题的。

gc删除map元素要这样写：
```go
var toDeleteKey []string  
for k := range kv.database {  
   if key2shard(k) == shard {  
      toDeleteKey = append(toDeleteKey, k)  
   }  
}  
for _, k := range toDeleteKey {  
   delete(kv.database, k)  
}
```

由于Follower不会去push，因此它不知道什么时候该gc。因此把gc写入日志是上策。

压缩rpc数量可能导致Challenge的要求很难满足，感觉很麻烦，暂不考虑。

sendMigrateShard也要检测kill。

测试开8线程限时3分钟刚刚好。
























