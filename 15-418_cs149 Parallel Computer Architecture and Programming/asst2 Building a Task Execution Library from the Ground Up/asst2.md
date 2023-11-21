
环境要求的服务器是8vCPU+32GB内存。


# Part A: Synchronous Bulk Task Launch

`run()` executes `num_total_tasks` instances of the specified task. Since this single function call results in the execution of many tasks, we refer to each call to `run()` as a _bulk task launch_.

step2时，任务队列领取到空了也不一定都完成了，可能才刚领取，因此主线程不得不等待所有任务已完成。

使用`queue<function<void()>>`的实现：
```
Executing test: super_super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_super_light
                                        STUDENT   REFERENCE   PERF?
[Serial]                                4.577     5.041       0.91  (OK)
[Parallel + Always Spawn]               85.449    103.686     0.82  (OK)
[Parallel + Thread Pool + Spin]         5.901     25.807      0.23  (OK)
[Parallel + Thread Pool + Sleep]        6.143     34.433      0.18  (OK)
================================================================================
Executing test: super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_light
                                        STUDENT   REFERENCE   PERF?
[Serial]                                54.517    65.396      0.83  (OK)
[Parallel + Always Spawn]               102.054   148.347     0.69  (OK)
[Parallel + Thread Pool + Spin]         21.052    32.26       0.65  (OK)
[Parallel + Thread Pool + Sleep]        35.873    38.296      0.94  (OK)
================================================================================
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                896.286   1082.361    0.83  (OK)
[Parallel + Always Spawn]               244.476   243.874     1.00  (OK)
[Parallel + Thread Pool + Spin]         892.575   199.9       4.47  (NOT OK)
[Parallel + Thread Pool + Sleep]        901.846   193.635     4.66  (NOT OK)
```

发现每次分配任务实际上分配的只是task id，因此完全可以只维护个递增的next_task_num变量，不需要再用queue了。

进入cv的带条件wait之前要看看是不是有必要进入，有时候条件已经被满足了。

去掉queue后：
```
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                880.017   1099.589    0.80  (OK)
[Parallel + Always Spawn]               234.721   243.891     0.96  (OK)
[Parallel + Thread Pool + Spin]         151.043   205.931     0.73  (OK)
[Parallel + Thread Pool + Sleep]        152.585   193.833     0.79  (OK)
```



