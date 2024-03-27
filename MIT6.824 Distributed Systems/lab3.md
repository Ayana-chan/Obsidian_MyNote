
lab3对心跳密度解禁，因此每次start时都可以心跳一次来及时发送请求，在网络情况较好的时候可以在很短时间内完成一次操作。

各个RPC之间不能互相信任，一个可能可以连续到达很多次，另一个可能连续到不了很多次，看起来就像是有一个走后门了一样。
## A

- Put() replaces the value for a particular key in the database
- Append(key, arg) appends arg to key's value
- Get() fetches the current value for a key

- A Get for a non-existant key should return an empty string
- An Append to a non-existant key should act like Put
- Each client talks to the service through a Clerk with Put/Append/Get methods
- A Clerk manages RPC interactions with the servers

You are allowed to add fields to the Raft ApplyMsg, and to add fields to Raft RPCs such as AppendEntries.

每个server绑定一个raft服务器，因此找leader是client的任务。

多个client向同一个具有leader的server发送请求，而此server要线性处理这些请求并返回给原client。

所有server都要监听applyCh并将它们应用到本地的kvdatabase里面。相当于raft赋予这个操作“可执行”权力，操作执行的地方依旧是kvserver。

发送的请求有可能在很久之后才被提交，那时候要考虑到。

It's OK to assume that a client will make only one call into a Clerk at a time.

带缓冲的channel可以防止在把applyCh的返回给分发的时候出现阻塞。

发现applyCh的时候当场判断是否重复并执行，然后通知原请求即可。

一个操作可能会被commit多次，但应当只执行一次。即使发现重复了，也不应该返回失败，而是返回成功但不执行。因此，去重的逻辑不应该写在请求上，而应该写在监听applyCh的应用处。这样即使服务器重启，重走这么多commit也不会出问题。（此时等待的channel里面塞什么似乎不重要？）（但是压缩时可能会出问题）

发送方不应该主动关channel，那就删除channel对应的map的key，让其不可达，并且可自然gc。（删map索引等价于删channel）。

可能有这样的及其刁钻的情况：判断超时，但尚未获得锁以删掉channel，于是applyCh的东西也会被加入到这个channel当中。但此时完全可以判定其为超时，不会影响任何东西。此时**带缓存的channel**的作用就凸显出来了，applyCh加入时不会阻塞，加完后可以带着数据被gc。

应当这样分解server的功能：
1. 若为Leader，接受client的请求，并将其发给Raft。
2. 所有服务器，监听applyCh，将新command应用于kv数据库。
3. 所有服务器，接收到command后通知正在等待回应的client。

读操作若被不同服务器执行两次，期间有一个写操作插队的话，可能导致两个读的结果不同；但它们都对应同一个请求，因此会随机返回其中一个。这不会破坏要求，依然满足线性一致性。

新leader当选后，老term的日志可能未提交，此时就需要手动给leader塞一个空log。但是，raft层直接加nil指令的log会导致lab2测试过不了（类型错误），因此让raft层当选leader后向kv层请求空日志，kv层收到后调用kv.rf.Start(nil)。

## B

You must revise your Raft code to operate while storing only the tail of the log. Raft should discard old log entries in a way that allows the Go garbage collector to free and re-use the memory; this requires that there be no reachable references (pointers) to the discarded log entries.

You should compare maxraftstate to persister.RaftStateSize(). Whenever your key/value server detects that the Raft state size is approaching this threshold, it should save a snapshot, and tell the Raft library that it has snapshotted, so that Raft can discard old log entries.

要保存Client的提交请求编号。

维护logOffset，表示snapshot之后日志的偏移量（此时所有操作只可能访问偏移量之后的日志）。原代码中所有下标访问都为逻辑访问（即逻辑上无视snapshot压缩），但访问的时候下标都得减去logOffset。

不要进行重复无意义甚至开倒车的snapshot操作。

先解决下标越界问题，再解决数据丢失问题。

永远注意，0不是“剩余日志”的一部分，而是快照的一部分。

leader重启后从后往前扫，所以任何重复传committed日志的AE必然是老RPC，其上的日志再多也是可以安全丢弃的。（待论证）

InstallSnapshot属于figure2的“RPC”范畴，因此也需要在发和收时都检测下台。

InstallSnapshot的lastIncludeIndex如果不比commitIndex大则直接丢弃。

kv层的apply拿到Snapshot后，如果它的lastIncludeIndex不大于lastappliedIndex则也丢掉。

installSnapshot传给kv的之前，可能有比lastIncludeIndex还靠后的log（raft层完成snapshot后获取到的新log）插队地apply到kv层，直接Snapshot覆盖的话超过lastIncludeIndex的log就可能丢失，但丢弃Snapshot的话会少东西。因此kv的apply接收也要注意顺序。因此，在kv接收apply的时候强制要求`kv.lastAppliedIndex+1 == applyMsg.CommandIndex`有序。
- 丢掉比Snapshot旧的日志不可能有问题
- 丢掉比Snapshot新的日志会出问题，因此raft层在向kv层通知完snapshot后，重新设置rf.lastApplied = args.LastIncludedIndex，来重新apply。

InstallSnapshot之后要改不少状态，如commitIndex、logOffset，如果有需要还要重置lastApplied。

时刻检测leader是否还是leader，很多下标越界就是这样造成的。

nextIndex如果根据响应到达的先后随意变化的话，可能导致其缩短到不超过logOffset，这样的nextIndex会引发各种问题。






