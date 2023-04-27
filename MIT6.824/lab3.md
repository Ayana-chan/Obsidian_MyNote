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

删除map的key后，value不一定真的释放了内存，因此需要手动关闭map中的channel，不过此时不需要上锁。

带缓冲的channel可以防止在把applyCh的返回给分发的时候出现阻塞。

发现applyCh的时候当场判断是否重复并执行，然后通知原请求即可。

一个操作可能会被commit多次，但应当只执行一次。即使发现重复了，也不应该返回失败，而是返回成功但不执行。因此，去重的逻辑不应该写在请求上，而应该写在监听applyCh的应用处。这样即使服务器重启，重走这么多commit也不会出问题。（此时等待的channel里面塞什么似乎不重要？）（但是压缩时可能会出问题）

发送方不应该主动关channel，那就删除channel对应的map的key，让其不可达，并且可自然gc。（删map索引等价于删channel。

可能有这样的及其刁钻的情况：判断超时，但尚未获得锁以删掉channel，于是applyCh的东西也会被加入到这个channel当中。但此时完全可以判定其为超时，不会影响任何东西。此时**带缓存的channel**的作用就凸显出来了，applyCh加入时不会阻塞，加完后可以带着数据被gc。






















