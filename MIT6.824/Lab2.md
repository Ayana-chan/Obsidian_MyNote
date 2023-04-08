- A service calls Make(peers,me,…) to create a Raft peer. The peers argument is an array of network identifiers of the Raft peers (including this one), for use with RPC. The me argument is the index of this peer in the peers array.
- Start(command) asks Raft to start the processing to append the command to the replicated log. Start() should return immediately, without waiting for the log appends to complete.
- The service expects your implementation to send an ApplyMsg for each newly committed log entry to the applyCh channel argument to Make().

## A 

heartbeats: AppendEntries RPCs with no log entries

- Make sure the election timeouts in different peers don't always fire at the same time, or else all peers will vote only for themselves and no one will become the leader.
- The tester requires that the leader send heartbeat RPCs no more than ten times per second.

- The tester calls your Raft's rf.Kill() when it is permanently shutting down an instance. You can check whether Kill() has been called using rf.killed(). You may want to do this in all loops, to avoid having dead Raft instances print confusing messages.

似乎断网后没办法向自己发rpc

只有follower能投票

成为follower时获得票（清空votedFor）

变成candidate后也获得的票，但只会留给自己

一轮投票失败，有一大群candidate。此时其中一个candidate开始拉票，它term++，这会导致其他candidate变成follower。这群follower会在自己的随机时间到达时正常变回candidate。

可能出现的网络延迟导致heartbeat延迟到达的问题（猜想）：若s1断网，s2与s3之间网速较慢。s2选举完获得了2、3票；此时s1网络恢复，并收到s2的heartbeat而变成follower；s3开始选举并且获得了1、3票。这就会导致s2和s3都变成leader。

leader通过心跳来控制term一致，也会别的旧leader下台；另外，还会比较返回值中的term让自己下台。

发现需要选举时，就得标记当前无leader，这必须是原子操作。
























