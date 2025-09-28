# saxpy

```txt
---------------------------------------------------------
Found 1 CUDA devices
Device 0: NVIDIA GeForce RTX 3060 Laptop GPU
   SMs:        30
   Global mem: 6144 MB
   CUDA Cap:   8.6
---------------------------------------------------------
Running 3 timing tests:
Effective BW by CUDA saxpy: 685.738 ms          [1.630 GB/s]
Kernel time costed by CUDA saxpy: 369.957 ms
Effective BW by CUDA saxpy: 300.589 ms          [3.718 GB/s]
Kernel time costed by CUDA saxpy: 4.348 ms
Effective BW by CUDA saxpy: 290.938 ms          [3.841 GB/s]
Kernel time costed by CUDA saxpy: 4.377 ms
```

# Prefix-Sum

![](assets/Pasted%20image%2020250928193545.png)

```txt
ayana@DESKTOP-2LR3H8Q:~/workspace/cs149_labs/asst3/scan$ ./cudaScan -n 99999
---------------------------------------------------------
Found 1 CUDA devices
Device 0: NVIDIA GeForce RTX 3060 Laptop GPU
   SMs:        30
   Global mem: 6144 MB
   CUDA Cap:   8.6
---------------------------------------------------------
Array size: 99999
Student GPU time: 0.238 ms
Scan outputs are correct!
```

1. 使用核函数能快速得出`repeat_p`数组，使得当`device_input[i] == device_input[i+1]`的时候，`repeat_p[i] == 1`。
2. 使用scan，可以将`repeat_p`转化为`repeat_count`，使得如果`repeat_p[i] == 1`，则`repeat_count[i]`表示这个i是第几个满足要求的。（具体来说，当`repeat_p[i] == 1`时，从`i+1`开始所有`repeat_count`都`+1`）
3. 因此，当`repeat_p[i] == 1`时，令`ans[repeat_count[i]] = i`，即可在`ans`中收集所有满足要求的下标。









