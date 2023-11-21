
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

去掉queue后，依然有两个测试无法完美通过：
```
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1548.974  1589.679    0.97  (OK)
[Parallel + Always Spawn]               390.673   319.226     1.22  (NOT OK)
[Parallel + Thread Pool + Spin]         1550.998  282.322     5.49  (NOT OK)
[Parallel + Thread Pool + Sleep]        242.71    264.911     0.92  (OK)
================================================================================
Executing test: recursive_fibonacci...
Reference binary: ./runtasks_ref_linux
Results for: recursive_fibonacci
                                        STUDENT   REFERENCE   PERF?
[Serial]                                794.64    1332.547    0.60  (OK)
[Parallel + Always Spawn]               147.338   201.821     0.73  (OK)
[Parallel + Thread Pool + Spin]         144.098   226.889     0.64  (OK)
[Parallel + Thread Pool + Sleep]        794.976   209.075     3.80  (NOT OK)
================================================================================
```

求斐波那契的时候，下标加一时，计算量就翻倍，线程间任务十分不均匀。此时，从大任务开始分配就瞬间快了好几倍。但这是硬编码。取巧，直接套用Fisher-Yates算法。这里感觉最好的实现是Distribute Work Queue，但太复杂了。

在sleep使用洗牌法后：
```
================================================================================
Running task system grading harness... (11 total tests)
  - Detected CPU with 8 execution contexts
  - Task system configured to use at most 8 threads
================================================================================
================================================================================
Executing test: super_super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_super_light
                                        STUDENT   REFERENCE   PERF?
[Serial]                                4.652     5.065       0.92  (OK)
[Parallel + Always Spawn]               87.196    100.703     0.87  (OK)
[Parallel + Thread Pool + Spin]         5.815     24.278      0.24  (OK)
[Parallel + Thread Pool + Sleep]        8.935     33.86       0.26  (OK)
================================================================================
Executing test: super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_light
                                        STUDENT   REFERENCE   PERF?
[Serial]                                52.841    68.165      0.78  (OK)
[Parallel + Always Spawn]               100.077   148.187     0.68  (OK)
[Parallel + Thread Pool + Spin]         13.93     30.734      0.45  (OK)
[Parallel + Thread Pool + Sleep]        34.968    38.418      0.91  (OK)
================================================================================
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                875.309   1099.207    0.80  (OK)
[Parallel + Always Spawn]               233.99    242.121     0.97  (OK)
[Parallel + Thread Pool + Spin]         155.875   207.017     0.75  (OK)
[Parallel + Thread Pool + Sleep]        157.834   195.452     0.81  (OK)
================================================================================
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1561.898  1602.294    0.97  (OK)
[Parallel + Always Spawn]               390.848   319.341     1.22  (NOT OK)
[Parallel + Thread Pool + Spin]         1562.523  291.004     5.37  (NOT OK)
[Parallel + Thread Pool + Sleep]        256.648   266.638     0.96  (OK)
================================================================================
Executing test: recursive_fibonacci...
Reference binary: ./runtasks_ref_linux
Results for: recursive_fibonacci
                                        STUDENT   REFERENCE   PERF?
[Serial]                                802.091   1344.305    0.60  (OK)
[Parallel + Always Spawn]               143.032   203.433     0.70  (OK)
[Parallel + Thread Pool + Spin]         144.66    230.229     0.63  (OK)
[Parallel + Thread Pool + Sleep]        122.334   208.137     0.59  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop
                                        STUDENT   REFERENCE   PERF?
[Serial]                                507.928   524.204     0.97  (OK)
[Parallel + Always Spawn]               545.968   677.81      0.81  (OK)
[Parallel + Thread Pool + Spin]         116.756   162.991     0.72  (OK)
[Parallel + Thread Pool + Sleep]        224.556   218.832     1.03  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fewer_tasks...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fewer_tasks
                                        STUDENT   REFERENCE   PERF?
[Serial]                                511.955   523.278     0.98  (OK)
[Parallel + Always Spawn]               527.5     584.175     0.90  (OK)
[Parallel + Thread Pool + Spin]         147.602   183.381     0.80  (OK)
[Parallel + Thread Pool + Sleep]        207.512   229.015     0.91  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fan_in...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fan_in
                                        STUDENT   REFERENCE   PERF?
[Serial]                                260.405   266.735     0.98  (OK)
[Parallel + Always Spawn]               102.651   98.06       1.05  (OK)
[Parallel + Thread Pool + Spin]         47.046    59.552      0.79  (OK)
[Parallel + Thread Pool + Sleep]        54.273    62.848      0.86  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_reduction_tree...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_reduction_tree
                                        STUDENT   REFERENCE   PERF?
[Serial]                                261.269   265.963     0.98  (OK)
[Parallel + Always Spawn]               54.232    54.053      1.00  (OK)
[Parallel + Thread Pool + Spin]         41.797    50.753      0.82  (OK)
[Parallel + Thread Pool + Sleep]        41.203    49.357      0.83  (OK)
================================================================================
Executing test: spin_between_run_calls...
Reference binary: ./runtasks_ref_linux
Results for: spin_between_run_calls
                                        STUDENT   REFERENCE   PERF?
[Serial]                                279.288   477.684     0.58  (OK)
[Parallel + Always Spawn]               280.157   239.814     1.17  (OK)
[Parallel + Thread Pool + Spin]         145.518   246.891     0.59  (OK)
[Parallel + Thread Pool + Sleep]        140.261   240.443     0.58  (OK)
================================================================================
Executing test: mandelbrot_chunked...
Reference binary: ./runtasks_ref_linux
Results for: mandelbrot_chunked
                                        STUDENT   REFERENCE   PERF?
[Serial]                                417.503   421.848     0.99  (OK)
[Parallel + Always Spawn]               56.185    56.583      0.99  (OK)
[Parallel + Thread Pool + Spin]         55.311    63.728      0.87  (OK)
[Parallel + Thread Pool + Sleep]        55.771    55.975      1.00  (OK)
================================================================================
Overall performance results
[Serial]                                : All passed Perf
[Parallel + Always Spawn]               : Perf did not pass all tests
[Parallel + Thread Pool + Spin]         : Perf did not pass all tests
[Parallel + Thread Pool + Sleep]        : All passed Perf
```

sleep全过，另外俩有一个不通过。spawn改成均匀静态分配、spin改成洗牌法之后，spin依旧没过，懒得改了：
```
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1562.08   1600.391    0.98  (OK)
[Parallel + Always Spawn]               328.835   317.137     1.04  (OK)
[Parallel + Thread Pool + Spin]         1569.004  282.488     5.55  (NOT OK)
[Parallel + Thread Pool + Sleep]        253.288   265.811     0.95  (OK)
```