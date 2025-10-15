
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
| 1000000         | 1.237           | 0.711           | 1.25            |
| 10000000        | 8.746           | 4.924           | 1.25            |
| 20000000        | 15.959          | 9.414           | 1.25            |
| 40000000        | 30.88           | 18.697          | 1.25            |
-------------------------------------------------------------------------
|                                   | Total score:    | 5.0/5.0         |
-------------------------------------------------------------------------
```

## repeat

1. 使用核函数能快速得出`repeat_p`数组，使得当`device_input[i] == device_input[i+1]`的时候，`repeat_p[i] == 1`。
2. 使用scan，可以将`repeat_p`转化为`repeat_count`，使得如果`repeat_p[i] == 1`，则`repeat_count[i]`表示这个i是第几个满足要求的。（具体来说，当`repeat_p[i] == 1`时，从`i+1`开始所有`repeat_count`都`+1`）
3. 因此，当`repeat_p[i] == 1`时，令`ans[repeat_count[i]] = i`，即可在`ans`中收集所有满足要求的下标。
	- 这里问题比较特殊，下标`len(repeat_p) - 1`总是为0。因此，直接使用`repeat_count[i] != repeat_count[i+1]`代替了`repeat_p[i] == 1`。

```txt
$ ./cudaScan -m find_repeats -i ones -n 9999
---------------------------------------------------------
Found 1 CUDA devices
Device 0: NVIDIA GeForce RTX 3060 Laptop GPU
   SMs:        30
   Global mem: 6144 MB
   CUDA Cap:   8.6
---------------------------------------------------------
Array size: 9999
Student GPU time: 0.203 ms
Find_repeats outputs are correct!
```

```txt
-------------------------
Find_repeats Score Table:
-------------------------
-------------------------------------------------------------------------
| Element Count   | Ref Time        | Student Time    | Score           |
-------------------------------------------------------------------------
| 1000000         | 2.244           | 0.94            | 1.25            |
| 10000000        | 14.665          | 9.438           | 1.25            |
| 20000000        | 28.524          | 16.87           | 1.25            |
| 40000000        | 55.197          | 31.936          | 1.25            |
-------------------------------------------------------------------------
|                                   | Total score:    | 5.0/5.0         |
-------------------------------------------------------------------------
```

# renderer

如果`#include <GL/glut.h>`报错，则安装依赖：
```bash
sudo apt install freeglut3-dev libgl1-mesa-dev libglu1-mesa-dev
```

为了跑ref实现，需要libglut.so.3文件，得软链接一下：
```shell
sudo ln -s /usr/lib/x86_64-linux-gnu/libglut.so.3.12.0 /usr/lib/x86_64-linux-gnu/libglut.so.3
```

checker.py默认调用的ref可执行文件名是`render_ref`，在linux上得换成`render_ref_x86`。

输入各个球体的3d左边、半径、颜色（RGB+alpha），要求渲染出一个指定分辨率的二维图片。
- 对二维图片里的每个像素，检查其涉及到的所有球体（point-in-circle tests），然后计算最终颜色。
- alpha是不透明度，1表示完全不透明，0则完全透明。

把颜色 `(C_r, C_g, C_b, C_alpha)` 渲染到当前背景色 `(P_r, P_g, P_b)` 的上方, 其算法为：
```txt
result_r = C_alpha * C_r + (1.0 - C_alpha) * P_r
result_g = C_alpha * C_g + (1.0 - C_alpha) * P_g
result_b = C_alpha * C_b + (1.0 - C_alpha) * P_b
```
该算法不满足交换律。renderer根据提供的球体顺序来进行渲染，越先提供的球体越靠后面。

一个像素（正方形）的颜色取决于其中心点的颜色。

circleBoxTest.cu_inl里面的函数提供了检查二维的圆形与矩形是否重合的设备函数：
- circleInBoxConservative：快速检查，但当返回为1时，有一定概率实际上不重合（例如圆刚好在矩形的角的斜外方）。
- circleInBox：完全精确计算是否重合，性能消耗较大。
可以先用circleInBoxConservative排除掉所有不重合的情况；如果重合了，再使用circleInBox做精确检查。

雪花场景是特殊计算的，每个球都是内深外浅。旧实现的场景判断写在最内循环，会拖慢速度。TODO：在最后再写到外部循环，利用模板。

float4：
```cpp
struct __align__(16) float4 {
    float x, y, z, w;
};
```

旧实现的Launch是一维的，每个block有256个线程。TODO：看16\*16好还是32\*32好

首次实现，每个pixel一个线程，每个线程都会顺序遍历所有circle。
![](assets/Pasted%20image%2020251015193736.png)

提供的scan必须要处理整个block，且不能处理block外的。

可以检测一个block和哪些球相关，把不相关的球直接剔除掉。目前想到的不同实现：
- launch一个新任务，一个球一个线程，每个线程里面都对所有32\*32的像素block进行box-circle相交检测。维护一个block数\*球数的global数组，来记录相交性。
- 依然使用原任务，整个block并发判断各个球是否涉及了当前block，然后利用scan把涉及的球都filter出来，再遍历这些球以计算颜色。为了记录相交性，需要申请的数组大小 相比其他实现 不会变化，但被分派到了各个shared memory当中，因此理论上效率更高。（采用此实现）

获取相交的circle数量时，需要读取scan结果的最后一项。
- 如果每个线程都计算数量的话，必须要先`__syncthreads()`来同步scan的结果，然后进行计算。
- 可以仅让最后一个线程进行计算，计算后再`__syncthreads()`同步计算结果。会有分支判断的开销。

没必要一次性计算完所有圆然后再渲染，计算完BLOCK_SIZE个球之后就渲染这些球。

首次达标：
![](assets/Pasted%20image%2020251015192658.png)

添加了circleInBox精确判断后。虽然看起来circleInBoxConservative的误判概率不大，但毕竟是并发加速了的，排除误判圆后也可以略微提升总体性能。
![](assets/Pasted%20image%2020251015192937.png)

使用模板去掉了运行时对flake场景的判断后：
![](assets/Pasted%20image%2020251015194851.png)


