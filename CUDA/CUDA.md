# 工程

使用saxpy介绍CUDA中与GPU的交互的基本写法：[An Easy Introduction to CUDA C and C++ | NVIDIA Technical Blog](https://developer.nvidia.com/blog/easy-introduction-cuda-c-and-c/)

核函数定义的时候，在最前面加上`__global__`即可。核函数内部可以直接访问`blockIdx`、`blockDim`、`threadIdx`等变量，它们都与具体的线程有关。
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


# 算法

scan，filter，flatten：
![](assets/c81fedf89341193172eaf478b87e1224.png)
