

两部分组成：
- a set of **replica groups**. Each replica group is responsible for a subset of the shards. A replica consists of a handful of servers that use Raft to replicate the group's shards. 
- **shard controller**. The shard controller decides which replica group should serve each shard; this information is called the **configuration**.

Clients consult the shard controller in order to find the replica group for a key, and replica groups consult the controller in order to find out what shards to serve. There is a single shard controller for the whole system, implemented as a fault-tolerant service using Raft.






























