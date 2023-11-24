
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

求斐波那契的时候，下标加一时，计算量就翻倍，线程间任务十分不均匀。此时，从大任务开始分配就瞬间快了好几倍。但这是硬编码。取巧，直接套用Fisher-Yates算法。这里感觉最好的实现是Distribute Work Queue，但太复杂了。这种**动态分配，但分配顺序被静态打乱**的方式算是我自己探索出的新方法。这个mapping可以被以任意的方式初始化。

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

sleep全过，另外俩有一个不通过。spawn改成均匀静态分配、spin改成洗牌法之后，spin依旧没过，而且ping_pong_equal也出问题了，甚是奇妙，可能是os调度问题，懒得改了：
```
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1045.147  1091.744    0.96  (OK)
[Parallel + Always Spawn]               260.121   245.19      1.06  (OK)
[Parallel + Thread Pool + Spin]         1041.817  202.348     5.15  (NOT OK)
[Parallel + Thread Pool + Sleep]        181.336   198.414     0.91  (OK)
================================================================================
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1555.266  1595.77     0.97  (OK)
[Parallel + Always Spawn]               336.504   321.626     1.05  (OK)
[Parallel + Thread Pool + Spin]         1559.995  285.991     5.45  (NOT OK)
[Parallel + Thread Pool + Sleep]        255.465   266.928     0.96  (OK)
```

wocker数量有问题，numThread个wocker以外，主线程也在干活，这不合理。为了实现方便，将主线程视为纯调度器。

发现时间不稳定的原因是finish没初始化，导致wocker有可能会直接退出，只有主线程在干活！把主线程干活的代码删了之后，直接没人干活了。

调整完毕后：
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
[Serial]                                4.573     5.154       0.89  (OK)
[Parallel + Always Spawn]               87.639    103.384     0.85  (OK)
[Parallel + Thread Pool + Spin]         12.845    25.606      0.50  (OK)
[Parallel + Thread Pool + Sleep]        29.244    33.637      0.87  (OK)
================================================================================
Executing test: super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_light
                                        STUDENT   REFERENCE   PERF?
[Serial]                                64.367    68.033      0.95  (OK)
[Parallel + Always Spawn]               99.719    148.403     0.67  (OK)
[Parallel + Thread Pool + Spin]         19.206    32.1        0.60  (OK)
[Parallel + Thread Pool + Sleep]        34.818    38.859      0.90  (OK)
================================================================================
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1060.756  1103.243    0.96  (OK)
[Parallel + Always Spawn]               257.926   248.035     1.04  (OK)
[Parallel + Thread Pool + Spin]         206.656   213.278     0.97  (OK)
[Parallel + Thread Pool + Sleep]        201.935   207.996     0.97  (OK)
================================================================================
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Serial]                                1566.967  1603.178    0.98  (OK)
[Parallel + Always Spawn]               340.589   325.77      1.05  (OK)
[Parallel + Thread Pool + Spin]         296.698   296.08      1.00  (OK)
[Parallel + Thread Pool + Sleep]        286.766   271.427     1.06  (OK)
================================================================================
Executing test: recursive_fibonacci...
Reference binary: ./runtasks_ref_linux
Results for: recursive_fibonacci
                                        STUDENT   REFERENCE   PERF?
[Serial]                                798.902   1345.464    0.59  (OK)
[Parallel + Always Spawn]               153.266   210.871     0.73  (OK)
[Parallel + Thread Pool + Spin]         152.331   247.375     0.62  (OK)
[Parallel + Thread Pool + Sleep]        137.192   217.174     0.63  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop
                                        STUDENT   REFERENCE   PERF?
[Serial]                                516.297   530.157     0.97  (OK)
[Parallel + Always Spawn]               548.502   683.785     0.80  (OK)
[Parallel + Thread Pool + Spin]         152.161   175.227     0.87  (OK)
[Parallel + Thread Pool + Sleep]        218.594   223.678     0.98  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fewer_tasks...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fewer_tasks
                                        STUDENT   REFERENCE   PERF?
[Serial]                                511.402   526.069     0.97  (OK)
[Parallel + Always Spawn]               534.214   579.837     0.92  (OK)
[Parallel + Thread Pool + Spin]         164.47    188.863     0.87  (OK)
[Parallel + Thread Pool + Sleep]        218.493   231.718     0.94  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fan_in...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fan_in
                                        STUDENT   REFERENCE   PERF?
[Serial]                                263.835   270.242     0.98  (OK)
[Parallel + Always Spawn]               101.409   100.78      1.01  (OK)
[Parallel + Thread Pool + Spin]         57.4      61.058      0.94  (OK)
[Parallel + Thread Pool + Sleep]        65.953    64.944      1.02  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_reduction_tree...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_reduction_tree
                                        STUDENT   REFERENCE   PERF?
[Serial]                                261.993   269.474     0.97  (OK)
[Parallel + Always Spawn]               53.672    54.468      0.99  (OK)
[Parallel + Thread Pool + Spin]         48.652    53.097      0.92  (OK)
[Parallel + Thread Pool + Sleep]        51.794    51.003      1.02  (OK)
================================================================================
Executing test: spin_between_run_calls...
Reference binary: ./runtasks_ref_linux
Results for: spin_between_run_calls
                                        STUDENT   REFERENCE   PERF?
[Serial]                                281.063   477.729     0.59  (OK)
[Parallel + Always Spawn]               143.189   241.231     0.59  (OK)
[Parallel + Thread Pool + Spin]         157.78    254.506     0.62  (OK)
[Parallel + Thread Pool + Sleep]        142.107   240.608     0.59  (OK)
================================================================================
Executing test: mandelbrot_chunked...
Reference binary: ./runtasks_ref_linux
Results for: mandelbrot_chunked
                                        STUDENT   REFERENCE   PERF?
[Serial]                                419.167   423.884     0.99  (OK)
[Parallel + Always Spawn]               55.978    57.588      0.97  (OK)
[Parallel + Thread Pool + Spin]         63.425    63.978      0.99  (OK)
[Parallel + Thread Pool + Sleep]        55.744    57.134      0.98  (OK)
================================================================================
Overall performance results
[Serial]                                : All passed Perf
[Parallel + Always Spawn]               : All passed Perf
[Parallel + Thread Pool + Spin]         : All passed Perf
[Parallel + Thread Pool + Sleep]        : All passed Perf
```

**似乎**所有mutex都是可删去的，单调的数据变化使得逻辑上conditional variable可能是不需要带锁的。注：atomic_int++是原子的且可以返回值，完全可以用来分发任务。
# Part B: 

 **your task runtime cannot begin execution of any task in the current bulk task launch until all tasks from the launches given in the dependency vector are complete!**

两个无依赖同时可执行的launch可以同时被执行，这样才能尽量塞满所有线程。因此，要把所有依赖满足的launch放在一起，全部可供获取task。

you can implement `run()` using appropriate calls to `runAsyncWithDeps()` and `sync()`.

sync后必须进行变量状态的清空，以应对下次runAsyncWithDeps。

- map1: launch id -> 所有依赖此launch的launch id
- map2: launch id -> 此launch的未完成的依赖个数
- map3: launch id -> 此launch的上下文
- set1: ready的launch id，可以从中找活干

由于执行过程中不会有新增launch，因此不需要顾及在launch完成后释放相关内存，做完后统一释放即可。

launch的task被领取完毕就可以踢出调度队伍。也因此，完成情况需要独立统计。

有任务时都开始，没任务时都停止，这也要求start相关逻辑要和找任务的逻辑绑定在同一个锁内。

测试发现，每个任务都较轻量级时，都出现了超时情况：
```
===================================================================================
Test name: super_super_light
===================================================================================
[Parallel + Thread Pool + Sleep]:               [35.916] ms
===================================================================================
ayana@ayana-ubuntu22:~/workspace/cs149_labs/asst2/part_b$ python3 ../tests/run_test_harness.py -a
================================================================================
Running task system grading harness... (22 total tests)
  - Detected CPU with 8 execution contexts
  - Task system configured to use at most 8 threads
================================================================================
================================================================================
Executing test: super_super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_super_light
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        35.305    33.288      1.06  (OK)
================================================================================
Executing test: super_super_light_async...
Reference binary: ./runtasks_ref_linux
Results for: super_super_light_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        24.091    12.537      1.92  (NOT OK)
================================================================================
Executing test: super_light...
Reference binary: ./runtasks_ref_linux
Results for: super_light
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        51.7      41.634      1.24  (NOT OK)
================================================================================
Executing test: super_light_async...
Reference binary: ./runtasks_ref_linux
Results for: super_light_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        44.722    33.275      1.34  (NOT OK)
================================================================================
Executing test: ping_pong_equal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        194.876   204.555     0.95  (OK)
================================================================================
Executing test: ping_pong_equal_async...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_equal_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        179.91    183.189     0.98  (OK)
================================================================================
Executing test: ping_pong_unequal...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        276.152   273.246     1.01  (OK)
================================================================================
Executing test: ping_pong_unequal_async...
Reference binary: ./runtasks_ref_linux
Results for: ping_pong_unequal_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        262.152   254.524     1.03  (OK)
================================================================================
Executing test: recursive_fibonacci...
Reference binary: ./runtasks_ref_linux
Results for: recursive_fibonacci
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        146.605   225.148     0.65  (OK)
================================================================================
Executing test: recursive_fibonacci_async...
Reference binary: ./runtasks_ref_linux
Results for: recursive_fibonacci_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        147.606   231.276     0.64  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        257.534   242.451     1.06  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_async...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        220.981   203.884     1.08  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fewer_tasks...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fewer_tasks
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        254.384   242.991     1.05  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fewer_tasks_async...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fewer_tasks_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        109.509   101.824     1.08  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fan_in...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fan_in
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        67.854    66.439      1.02  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_fan_in_async...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_fan_in_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        47.429    49.931      0.95  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_reduction_tree...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_reduction_tree
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        49.424    52.152      0.95  (OK)
================================================================================
Executing test: math_operations_in_tight_for_loop_reduction_tree_async...
Reference binary: ./runtasks_ref_linux
Results for: math_operations_in_tight_for_loop_reduction_tree_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        45.305    44.934      1.01  (OK)
================================================================================
Executing test: spin_between_run_calls...
Reference binary: ./runtasks_ref_linux
Results for: spin_between_run_calls
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        143.815   240.627     0.60  (OK)
================================================================================
Executing test: spin_between_run_calls_async...
Reference binary: ./runtasks_ref_linux
Results for: spin_between_run_calls_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        143.054   239.854     0.60  (OK)
================================================================================
Executing test: mandelbrot_chunked...
Reference binary: ./runtasks_ref_linux
Results for: mandelbrot_chunked
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        54.568    55.066      0.99  (OK)
================================================================================
Executing test: mandelbrot_chunked_async...
Reference binary: ./runtasks_ref_linux
Results for: mandelbrot_chunked_async
                                        STUDENT   REFERENCE   PERF?
[Parallel + Thread Pool + Sleep]        55.696    55.446      1.00  (OK)
================================================================================
Overall performance results
[Parallel + Thread Pool + Sleep]        : Perf did not pass all tests
```

通过打印发现，某个线程会连续得到很多次start_mu锁，然后才会让给别的线程（即某个线程会连续执行许多次任务，而其他线程闲置）。在yourtest里面测试了一下，如果每个task比较繁重的话，就不会出现这种情况。在实体机上试验发现确实有可能单线程一直占据mutex：
```cpp
int main() {  
    std::mutex mu;  
    std::vector<int> res(8);  
    int value = 0;  
  
    auto thread_func = [&](int id) {  
        for (int _i = 0; _i < 20; _i++) {  
            {  
                std::cout << "th " << id << " try get lock\n";  
                std::unique_lock<std::mutex> lock(mu, std::defer_lock);  
                std::cout << "th " << id << " got lock\n";  
                value++;  
            }  
            res[id] += value;  
        }  
    };  
  
    std::vector<std::thread> lt(8);  
    for (int i = 0; i < 8; i++) {  
        lt[i] = std::thread(thread_func, i);  
    }  
  
    for (int i = 0; i < 8; i++) {  
        lt[i].join();  
    }  
  
    for(auto &r: res){  
        std::cout<<r<<" ";  
    }  
    std::cout<<"\n";  
  
    return 0;  
}
```

想要解决此问题：
1. 增加任务获取的粒度，从而降低锁消耗。
2. 将同步机制优化，比如使用原子、避免加大锁。

试了下使用原子+CAS进行任务分配，结果依然不变。

试了下每次分配多个任务，结果依然不变。

优化了一下task func的传递，从返回闭包改成返回仿函数，似乎略有性能提升，但并没有大进步。



