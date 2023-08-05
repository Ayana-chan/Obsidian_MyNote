- A service calls Make(peers,me,…) to create a Raft peer. The peers argument is an array of network identifiers of the Raft peers (including this one), for use with RPC. The me argument is the index of this peer in the peers array.
- Start(command) asks Raft to start the processing to append the command to the replicated log. Start() should return immediately, without waiting for the log appends to complete.
- The service expects your implementation to send an ApplyMsg for each newly committed log entry to the applyCh channel argument to Make().

一定要丢弃旧term的RPC响应！！！否则会让一些逻辑十分麻烦，甚至拖慢时间导致lab2C巨折磨！！！

Make的时候确保全部变量初始化完了才开始计时，或者说变量赋值完了才能触发事件，否则在make数量（压力）很大的时候会导致没make完就开始选举，以至于报race。（这个race只在lab4B出现过）
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

之所以votedFor为candidateId时再投一次给它，是因为RPC发失败了要重发。**似乎没写重发逻辑。**
## B

测试时不要开太多线程，超两秒未提交某一项就会不通过。不要让CPU压力爆满（在lab3里面没有超时问题所以可以尽情上压力）。

**日志0是不会被提交的。**

RPC网络完全不可信任，每次处理响应的时候都要看看是否state变化，日志是否被删改（比如比较AE的最后一个日志与当前日志的下标任期是否对得上）。

applyCh是慢速的，如果两次commit非常接近，又同时塞入applyCh，就可能导致第二次的日志插队，导致order错误。

一个新leader当选时，即使没有新日志，它也会因为nextIndex为log的长度+1而可以通过preIndex和preTerm去检测follower日志是否和其完全相同，若不相同则马上开始发日志。

**commit了的日志是不会被删的**，因此可以放心地去apply。

apply任务必须是单线程的，使用channel进行保证，不然的话由于commitIndex是在apply任务前变的，可能导致乱序提交。



#### 下面的bug可能只有在lab3里面才能发现，因为lab2稍微加点压力就先爆超时了。

---
原理解：
>AE在高并发测试中有可能错位到达导致最终日志长度小于commitIndex（短的AE先发出后到位）。现实中应该很难发生，但以防万一让apply时不得超过日志长度。但是，虽然错位的返回可以让matchIndex和nextIndex符合最后一次的AE，但commitIndex是禁止缩短的，这可能导致commit项丢失。

情景：
>Leader: 12345  
>Follower: 12  
>
>Leader发送345，但网络延迟
>Leader收到新日志67
>Leader: 1234567  
>
>Leader发送34567,直接到达  
>Follower:1234567  
>
>此时345再到达  
>Follower: 1234567还是12345


figure2中AppendEntries RPC的第三点：
>If an existing entry conflicts with a new one (same indexbut different terms), delete the existing entry and all that follow it (§5.3)

因此确实没冲突就别删，应该为1234567

---

leader发送heartBeat时如果没有原子性地检测state的话，如果发送前下台了，可能日志也会发生变化。看似没有副作用，但此时再用nextIndex去获取日志项的时候就可能发生下标越界。

---

rpc报race，不知道什么原因：

```go
func (rf *Raft) sendAppendEntries(server int, args *AppendEntriesArgs, reply *AppendEntriesReply) bool {  
	ok := rf.peers[server].Call("Raft.AppendEntries", args, reply) //该行会报一个莫名其妙的race  
	return ok  
}
```

```bash
==================
WARNING: DATA RACE
Read at 0x00c0006713e0 by goroutine 132:
  encoding/gob.encInt()
      /usr/local/go/src/reflect/value.go:983 +0x1df
  encoding/gob.(*Encoder).encodeStruct()
      /usr/local/go/src/encoding/gob/encode.go:328 +0x436
  encoding/gob.encOpFor.func4()
      /usr/local/go/src/encoding/gob/encode.go:581 +0xf0
  encoding/gob.(*Encoder).encodeArray()
      /usr/local/go/src/encoding/gob/encode.go:351 +0x26f
  encoding/gob.encOpFor.func1()
      /usr/local/go/src/encoding/gob/encode.go:551 +0x1a3
  encoding/gob.(*Encoder).encodeStruct()
      /usr/local/go/src/encoding/gob/encode.go:328 +0x436
  encoding/gob.(*Encoder).encode()
      /usr/local/go/src/encoding/gob/encode.go:701 +0x1fe
  encoding/gob.(*Encoder).EncodeValue()
      /usr/local/go/src/encoding/gob/encoder.go:251 +0x666
  encoding/gob.(*Encoder).Encode()
      /usr/local/go/src/encoding/gob/encoder.go:176 +0x5b
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labgob.(*LabEncoder).Encode()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labgob/labgob.go:34 +0x7b
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc.(*ClientEnd).Call()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc/labrpc.go:93 +0x1a4
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/raft.(*Raft).sendHeartBeats.func1()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/raft/raft.go:427 +0x6be

Previous write at 0x00c0006713e0 by goroutine 186:
  [failed to restore the stack]

Goroutine 132 (running) created at:
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/raft.(*Raft).sendHeartBeats()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/raft/raft.go:443 +0x319

Goroutine 186 (running) created at:
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc.(*Network).processReq()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc/labrpc.go:237 +0x174
==================
==================
WARNING: DATA RACE
Read at 0x00c0006713e8 by goroutine 132:
  encoding/gob.encOpFor.func5()
      /usr/local/go/src/reflect/value.go:1078 +0x218
  encoding/gob.(*Encoder).encodeStruct()
      /usr/local/go/src/encoding/gob/encode.go:328 +0x436
  encoding/gob.encOpFor.func4()
      /usr/local/go/src/encoding/gob/encode.go:581 +0xf0
  encoding/gob.(*Encoder).encodeArray()
      /usr/local/go/src/encoding/gob/encode.go:351 +0x26f
  encoding/gob.encOpFor.func1()
      /usr/local/go/src/encoding/gob/encode.go:551 +0x1a3
  encoding/gob.(*Encoder).encodeStruct()
      /usr/local/go/src/encoding/gob/encode.go:328 +0x436
  encoding/gob.(*Encoder).encode()
      /usr/local/go/src/encoding/gob/encode.go:701 +0x1fe
  encoding/gob.(*Encoder).EncodeValue()
      /usr/local/go/src/encoding/gob/encoder.go:251 +0x666
  encoding/gob.(*Encoder).Encode()
      /usr/local/go/src/encoding/gob/encoder.go:176 +0x5b
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labgob.(*LabEncoder).Encode()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labgob/labgob.go:34 +0x7b
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc.(*ClientEnd).Call()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc/labrpc.go:93 +0x1a4
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/raft.(*Raft).sendHeartBeats.func1()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/raft/raft.go:427 +0x6be

Previous write at 0x00c0006713e8 by goroutine 186:
  [failed to restore the stack]

Goroutine 132 (running) created at:
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/raft.(*Raft).sendHeartBeats()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/raft/raft.go:443 +0x319

Goroutine 186 (running) created at:
  _/root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc.(*Network).processReq()
      /root/AAAAA/lab6.824/LabSoftware6.824/src/labrpc/labrpc.go:237 +0x174
==================
```

原因似乎找到了：

```go
//这是浅拷贝，会引发race（“rpc编码”与“修改log”之间的读写race）
args.Entries = rf.log[rf.nextIndex[s]:]  
//使用append达成深拷贝，从而让args真正独立开来
args.Entries = append(args.Entries, rf.log[rf.nextIndex[s]:]...)
```

---

# C

超时过于频繁就看看有没有写fast backup，以及是否尽快丢弃旧term的log。

持久化理由：
- votedFor： 防止投票两次出现脑裂。
- term：防止leader在网络隔离后，其他服务器由于宕机重启忘掉了term以至于投票出了小term的leader，致使原leader回归后起日志与其他服务器的冲突。
- log： 把commit的都存储了，以确保其绝对存储了多数。













