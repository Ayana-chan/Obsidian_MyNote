

# Program 1: Parallel Fractal Generation Using Threads

8个CPU核，且没hyperthread。

```bash
./mandelbrot --threads 4
```

view1:

| numThreads | speedup |
| ---------- | ------- |
| 2          | 2.01    |
| 3          | 1.63    |
| 4          | 2.43    |
| 5          | 2.48    |
| 6          | 3.25    |
| 7          | 3.26    |
| 8          | 3.83    |
| 16         | 5.11    | 

```
Thread 0 finish work in 0.008 ms
Thread 7 finish work in 0.008 ms
Thread 6 finish work in 0.036 ms
Thread 1 finish work in 0.039 ms
Thread 2 finish work in 0.078 ms
Thread 5 finish work in 0.080 ms
Thread 3 finish work in 0.118 ms
Thread 4 finish work in 0.119 ms
```

view2:

```
Thread 5 finish work in 0.031 ms
Thread 2 finish work in 0.033 ms
Thread 6 finish work in 0.034 ms
Thread 3 finish work in 0.036 ms
Thread 4 finish work in 0.037 ms
Thread 7 finish work in 0.039 ms
Thread 1 finish work in 0.048 ms
Thread 0 finish work in 0.065 ms
```

**线程花费时间不均衡的原因是每一行的计算量是不固定的。**

线程轮流一行一行地取，这样可以做到几乎平均分配，但8线程最多加速6.62倍。

view1：

```
Thread 6 finish work in 0.064 ms
Thread 0 finish work in 0.065 ms
Thread 3 finish work in 0.070 ms
Thread 5 finish work in 0.076 ms
Thread 2 finish work in 0.095 ms
Thread 7 finish work in 0.104 ms
Thread 1 finish work in 0.108 ms
Thread 4 finish work in 0.111 ms
```

view2：

```
Thread 0 finish work in 0.039 ms
Thread 2 finish work in 0.041 ms
Thread 5 finish work in 0.042 ms
Thread 4 finish work in 0.045 ms
Thread 1 finish work in 0.045 ms
Thread 3 finish work in 0.047 ms
Thread 6 finish work in 0.053 ms
Thread 7 finish work in 0.051 ms
```

TODO：继续优化or换个平台

# Program 2: Vectorizing Code Using SIMD Intrinsics

`./myexp -s 3 -l`：

```
CLAMPED EXPONENT (required) 
Results matched with answer!
***************** Printing Vector Unit Execution Log *****************
 Instruction | Vector Lane Occupancy ('*' for active, '_' for inactive)
------------- --------------------------------------------------------
        vset | ****
        vset | ****
        vset | ****
        vset | ****
       vload | ***_
       vload | ***_
         veq | ***_
      vstore | ____
     masknot | ****
     maskand | ****
       vload | ***_
        vsub | ***_
         veq | ***_
     cntbits | ****
     cntbits | ****
     masknot | ****
     maskand | ****
       vmult | ***_
        vsub | ***_
         veq | ***_
     cntbits | ****
     masknot | ****
     maskand | ****
       vmult | ***_
        vsub | ***_
         veq | ***_
     cntbits | ****
     masknot | ****
     maskand | ****
       vmult | ***_
        vsub | ***_
         veq | ***_
     cntbits | ****
     masknot | ****
     maskand | ****
       vmult | ***_
        vsub | ***_
         veq | ***_
     cntbits | ****
     masknot | ****
     maskand | ****
       vmult | *___
        vsub | *___
         veq | *___
     cntbits | ****
         vgt | ***_
        vset | ***_
      vstore | ***_
****************** Printing Vector Unit Statistics *******************
Vector Width:              4
Total Vector Instructions: 48
Vector Utilization:        82.3%
Utilized Vector Lanes:     158
Total Vector Lanes:        192
************************ Result Verification *************************
Passed!!!
```

`./myexp -s 10000`：

```
Vector Width:              2
Total Vector Instructions: 224090
Vector Utilization:        84.8%
Utilized Vector Lanes:     379989
Total Vector Lanes:        448180
```

```
Vector Width:              4
Total Vector Instructions: 131862
Vector Utilization:        79.6%
Utilized Vector Lanes:     419627
Total Vector Lanes:        527448
```

```
Vector Width:              8
Total Vector Instructions: 72440
Vector Utilization:        76.9%
Utilized Vector Lanes:     445671
Total Vector Lanes:        579520
```

```
Vector Width:              16
Total Vector Instructions: 37950
Vector Utilization:        75.7%
Utilized Vector Lanes:     459527
Total Vector Lanes:        607200
```

vector长度过长的时候，很多指令（特别是count循环处）会有特别多空操作。

# Program 3: Parallel Fractal Generation Using ISPC


















