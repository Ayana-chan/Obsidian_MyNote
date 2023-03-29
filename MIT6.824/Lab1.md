
Master只分配工作，Worker实际处理工作。

- The map phase should divide the intermediate keys into buckets for nReduce reduce tasks, where nReduce is the argument that main/mrmaster.go passes to MakeMaster(). **Done**
- The worker implementation should put the output of the X'th reduce task in the file mr-out-X.
- A mr-out-X file should contain one line per Reduce function output. The line should be generated with the Go "%v %v" format, called with the key and value. Have a look in main/mrsequential.go for the line commented "this is the correct format". The test script will fail if your implementation deviates too much from this format.
- You can modify mr/worker.go, mr/master.go, and mr/rpc.go. You can temporarily modify other files for testing, but make sure your code works with the original versions; we'll test with the original versions.
- The worker should put intermediate Map output in files in the current directory, where your worker can later read them as input to Reduce tasks. Map的输出存到json文件里面，作为reduce的输入。去`mrsequential.go`偷
- main/mrmaster.go expects mr/master.go to implement a Done() method that returns true when the MapReduce job is completely finished; at that point, mrmaster.go will exit.
- When the job is completely finished, the worker processes should exit. A simple way to implement this is to use the return value from call(): if the worker fails to contact the master, it can assume that the master has exited because the job is done, and so the worker can terminate too. Depending on your design, you might also find it helpful to have a "please exit" pseudo-task that the master can give to workers.
- The master can't reliably distinguish between crashed workers, workers that are alive but have stalled for some reason, and workers that are executing but too slowly to be useful. The best you can do is have the master wait for some amount of time, and then give up and re-issue the task to a different worker. For this lab, have the master wait for ten seconds; after that the master should assume the worker has died (of course, it might not have). 给Worker十秒钟界限，若超出十秒未返回则重新派发请求。注意避免过慢的Worker在后来返回可能出现的问题。
- To ensure that nobody observes partially written files in the presence of crashes, the MapReduce paper mentions the trick of using a temporary file and atomically renaming it once it is completely written. You can use ioutil.TempFile to create a temporary file and os.Rename to atomically rename it. 临时文件，完成任务后重命名
- A reasonable naming convention for intermediate files is mr-X-Y, where X is the Map task number, and Y is the reduce task number.

用ihash将key变成哈希值，并根据哈希值对总reduce数的取余来决定将该key-value交给哪个reduce去处理（同词必然对应同reduce）。

>use ihash(key) % NReduce to choose the reduce


只有master有race。

reduces can't start until the last map has finished 

确认完成的时候用三次握手？


没用channel！！！！！

~~改成使用sync.Cond实现任务请求，每个请求失败时都会sleep一段时间再解锁，这样可以让全部worker都在同一个固定时间间隔队列上排队请求，而不是各自都高频遍历。~~

最佳实现是使用双向链表（List）！

























