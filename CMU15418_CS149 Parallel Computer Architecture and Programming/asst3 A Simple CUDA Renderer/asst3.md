
环境：
- NVIDIA GeForce RTX 3060 Laptop GPU
- CUDA Toolkit 12.9
- wsl2 Ubuntu 24.04

```txt
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Tue_May_27_02:21:03_PDT_2025
Cuda compilation tools, release 12.9, V12.9.86
Build cuda_12.9.r12.9/compiler.36037853_0
```

```txt
$ nvidia-smi
Mon Sep 29 15:59:13 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.82.10              Driver Version: 581.29         CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3060 ...    On  |   00000000:01:00.0  On |                  N/A |
| N/A   68C    P8             16W /  125W |    1355MiB /   6144MiB |     17%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```
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
Effective BW by CUDA saxpy: 354.585 ms          [3.152 GB/s]
Kernel time costed by CUDA saxpy: 98.335 ms
Effective BW by CUDA saxpy: 252.670 ms          [4.423 GB/s]
Kernel time costed by CUDA saxpy: 4.135 ms
Effective BW by CUDA saxpy: 268.717 ms          [4.159 GB/s]
Kernel time costed by CUDA saxpy: 4.208 ms
```

# Prefix-Sum

## scan

![](assets/Pasted%20image%2020250928193545.png)

```txt
$ ./cudaScan -n 99999
---------------------------------------------------------
Found 1 CUDA devices
Device 0: NVIDIA GeForce RTX 3060 Laptop GPU
   SMs:        30
   Global mem: 6144 MB
   CUDA Cap:   8.6
---------------------------------------------------------
Array size: 99999
Student GPU time: 0.247 ms
Scan outputs are correct!
```

```txt
-------------------------
Scan Score Table:
-------------------------
-------------------------------------------------------------------------
| Element Count   | Ref Time        | Student Time    | Score           |
-------------------------------------------------------------------------
| 1000000         | 1.236           | 0.697           | 1.25            |
| 10000000        | 8.655           | 4.844           | 1.25            |
| 20000000        | 15.813          | 9.426           | 1.25            |
| 40000000        | 30.838          | 18.69           | 1.25            |
-------------------------------------------------------------------------
|                                   | Total score:    | 5.0/5.0         |
-------------------------------------------------------------------------
```

## repeat

1. 使用核函数能快速得出`repeat_p`数组，使得当`device_input[i] == device_input[i+1]`的时候，`repeat_p[i] == 1`。
2. 使用scan，可以将`repeat_p`转化为`repeat_count`，使得如果`repeat_p[i] == 1`，则`repeat_count[i]`表示这个i是第几个满足要求的。（具体来说，当`repeat_p[i] == 1`时，从`i+1`开始所有`repeat_count`都`+1`）
3. 因此，当`repeat_p[i] == 1`时，令`ans[repeat_count[i]] = i`，即可在`ans`中收集所有满足要求的下标。








