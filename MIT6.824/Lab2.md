- A service calls Make(peers,me,…) to create a Raft peer. The peers argument is an array of network identifiers of the Raft peers (including this one), for use with RPC. The me argument is the index of this peer in the peers array.
- Start(command) asks Raft to start the processing to append the command to the replicated log. Start() should return immediately, without waiting for the log appends to complete.
- The service expects your implementation to send an ApplyMsg for each newly committed log entry to the applyCh channel argument to Make().

## A 

heartbeats: AppendEntries RPCs with no log entries

- Make sure the election timeouts in different peers don't always fire at the same time, or else all peers will vote only for themselves and no one will become the leader.
- The tester requires that the leader send heartbeat RPCs no more than ten times per second.

- The tester calls your Raft's rf.Kill() when it is permanently shutting down an instance. You can check whether Kill() has been called using rf.killed(). You may want to do this in all loops, to avoid having dead Raft instances print confusing messages.

成为follower时获得票（清空votedFor）

变成candidate后也获得票，但只会留给自己

一个candidate出现后，向其他follower求票并更新它们的时钟。如果它未当选，则要么它时钟先到再次选举，要么某个follower时钟先到变成candidate然后让原candidate变成follower。

leader通过心跳来控制term一致，也会别的旧leader下台；另外，还会比较返回值中的term让自己下台。

发现需要选举时，就得标记当前无leader，这必须是原子操作。

要十分注意是否在发现高term时更新自己的term（无论任何身份）。这影响到是否有不必要的leader发送，以及是否能让选举正常进行（不更新的话，各个candidate和follower的任期可能会一直错开，导致一直重置选举）。
## B

测试时不要开太多线程，超两秒未提交某一项就会不通过。不要让CPU压力爆满。

RPC网络完全不可信任，每次处理响应的时候都要看看是否state变化，日志是否被删改（比如比较AE的最后一个日志与当前日志的下标任期是否对得上）。

applyCh是慢速的，如果两次commit非常接近，又同时塞入applyCh，就可能导致第二次的日志插队，导致order错误。

一个新leader当选时，即使没有新日志，它也会因为nextIndex为log的长度+1而可以通过preIndex和preTerm去检测follower日志是否和其完全相同，若不相同则马上开始发日志。


















