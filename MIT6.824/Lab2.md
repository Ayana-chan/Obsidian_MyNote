- A service calls Make(peers,me,…) to create a Raft peer. The peers argument is an array of network identifiers of the Raft peers (including this one), for use with RPC. The me argument is the index of this peer in the peers array.
- Start(command) asks Raft to start the processing to append the command to the replicated log. Start() should return immediately, without waiting for the log appends to complete.
- The service expects your implementation to send an ApplyMsg for each newly committed log entry to the applyCh channel argument to Make().

## A 

heartbeats: AppendEntries RPCs with no log entries

- Make sure the election timeouts in different peers don't always fire at the same time, or else all peers will vote only for themselves and no one will become the leader.
- The tester requires that the leader send heartbeat RPCs no more than ten times per second.

- The tester calls your Raft's rf.Kill() when it is permanently shutting down an instance. You can check whether Kill() has been called using rf.killed(). You may want to do this in all loops, to avoid having dead Raft instances print confusing messages.