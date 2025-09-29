# 安装

总共有这几个东西要注意安装版本正确：
- 显卡驱动
- CUDA
- CUDA toolkit

安装好toolkit后，在`./bashrc`里面配置nvcc路径：
```bash
# CUDA toolkit
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

## 多版本toolkit

安装多个版本后，`/usr/local`下可能会包含以下这些目录：
```
cuda cuda-12 cuda-12.9 cuda-13 cuda-13.0
```
它们互相通过软链接来表达“默认”语义。
- 例如，一些程序的makefile指定的链接：`-L/usr/local/cuda/lib64/ -lcudart`，表示链接当前系统内配置的默认cuda版本，例如可能是13.0。
- 配置nvcc环境变量的时候，使用的是`/usr/local/cuda/bin`，表示使用当前系统的默认cuda版本下的nvcc。

直接使用update-alternatives来切换默认cuda版本：
```bash
/usr/local$ sudo update-alternatives --config cuda
There are 2 choices for the alternative cuda (providing /usr/local/cuda).

  Selection    Path                  Priority   Status
------------------------------------------------------------
* 0            /usr/local/cuda-13.0   130       auto mode
  1            /usr/local/cuda-12.9   129       manual mode
  2            /usr/local/cuda-13.0   130       manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/local/cuda-12.9 to provide /usr/local/cuda (cuda) in manual mode
```

如果只是编译好了的二进制文件在**运行时**需要链接特定版本的CUDA库，则下载安装对应版本的toolkit就行，**不需要切换版本**。

## wsl

wsl会使用Windows的CUDA driver，而一般的CUDA toolkit会自带CUDA driver并进行覆盖，这会造成wsl的CUDA出错。因此，安装wsl的时候，要指定使用wsl版本的toolkit，会保证不覆盖driver：[CUDA Toolkit 12.9 Update 1 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-12-9-1-download-archive?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local)。

# 工程

使用saxpy介绍CUDA中与GPU的交互的基本写法：[An Easy Introduction to CUDA C and C++ | NVIDIA Technical Blog](https://developer.nvidia.com/blog/easy-introduction-cuda-c-and-c/)

核函数定义的时候，在最前面加上`__global__`（可被CPU调用）或`__device__`（只能被其他核函数调用）即可。核函数内部可以直接访问`blockIdx`、`blockDim`、`threadIdx`等变量，它们都与具体的线程有关。
```cpp
__global__ void  
judge_repeat_kernel(int* data, const int length, int* result) {  
    // 一维情况下的下标计算
    int index = blockIdx.x * blockDim.x + threadIdx.x;  
    // ...  
}
```

核函数调用的时候，函数参数依然是普通的参数，而于CUDA相关的参数都写在`<<<>>>`里面。第一个参数是block数量，第二个参数是每个block里面的thread数量。
```cpp
func_kernel<<<blocks, THREADS_PER_BLOCK>>>(data, length, result);
```

因为核函数的启动是以block为单位的，因此被启动的线程数量不一定和所需的线程数量相对应，所以一般都要在核函数参数里面加上边界信息（如`length`），然后在index超出边界的时候（如`index >= length`）不执行任何操作。

kernel的装载是异步的，可以使用`cudaDeviceSynchronize()`来等待所有CUDA任务完成。
- cudaMemcpy总是会等待对应任务完成。
- 多个CUDA任务之间按照调用顺序完成任务。
因此，大部分情况下不需要特地同步。

debug的时候使用的宏，可以包裹在cudaGetLastError或cudaMemcpy的外面：
```cpp
#define CHECK(call) \  
    do { \  
        cudaError_t err = call; \  
        if (err != cudaSuccess) { \  
            std::cerr << "Error: " << cudaGetErrorString(err) << " in file " << __FILE__ << " at line " << __LINE__ << std::endl; \  
            exit(EXIT_FAILURE); \  
        } \  
    } while (0)
```
使用示例：
```cpp
exclusive_scan_upsweep_kernel<<<blocks, THREADS_PER_BLOCK>>>(result, two_d, iter_time);  
cudaDeviceSynchronize();  
CHECK(cudaGetLastError());
```
进一步封装：
```cpp
// turn on/off debug output  
#define SYNC_CHECK_DEBUG_ON 0  
  
#if SYNC_CHECK_DEBUG_ON == 1  
#define SYNC_CHECK_DEBUG() do { \  
cudaDeviceSynchronize(); \  
CHECK(cudaGetLastError()); \  
} while (0)  
#else  
#define SYNC_CHECK_DEBUG() do {} while (0)  
#endif
```

# 算法

scan，filter，flatten：
![](assets/c81fedf89341193172eaf478b87e1224.png)
