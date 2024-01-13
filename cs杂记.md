# 链接

[计算机教育中缺失的一课 · the missing semester of your cs education](https://missing-semester-cn.github.io/)

[GitHub - youngyangyang04/leetcode-master: 《代码随想录》LeetCode 刷题攻略：200道经典题目刷题顺序，共60w字的详细图解，视频难点剖析，50余张思维导图，支持C++，Java，Python，Go，JavaScript等多语言版本，从此算法学习不再迷茫！🔥🔥 来看看，你会发现相见恨晚！🚀](https://github.com/youngyangyang04/leetcode-master)

# pasteme

可以管理、分享文本，可以用docker快速部署以搭建属于自己的服务。

[PasteMe Docs](https://docs.pasteme.cn/#/deploy/docker)
# parameter和argument

parameter: 定义时的参数（形参）。

argument: 调用时传入的参数（实参）。
# 用于提高代码性能，简单好用的性能分析工具有哪些?

性能分析方面相当有用和简单工具是[print timing](https://missing-semester-cn.github.io/2020/debugging-profiling/#timing)。你只需手动计算代码不同部分之间花费的时间。通过重复执行此操作，你可以有效地对代码进行二分法搜索，并找到花费时间最长的代码段。

对于更高级的工具， Valgrind 的 [Callgrind](http://valgrind.org/docs/manual/cl-manual.html)可让你运行程序并计算所有的时间花费以及所有调用堆栈（即哪个函数调用了另一个函数）。然后，它会生成带注释的代码版本，其中包含每行花费的时间。但是，它会使程序运行速度降低一个数量级，并且不支持线程。其他的， [`perf`](http://www.brendangregg.com/perf.html) 工具和其他特定语言的采样性能分析器可以非常快速地输出有用的数据。[Flamegraphs](http://www.brendangregg.com/flamegraphs.html) 是对采样分析器结果的可视化工具。你还可以使用针对特定编程语言或任务的工具。例如，对于 Web 开发而言，Chrome 和 Firefox 内置的开发工具具有出色的性能分析器。

有时，代码中最慢的部分是系统等待磁盘读取或网络数据包之类的事件。在这些情况下，需要检查根据硬件性能估算的理论速度是否不偏离实际数值，也有专门的工具来分析系统调用中的等待时间，包括用于用户程序内核跟踪的[eBPF](http://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html) 。如果需要低级的性能分析， [`bpftrace`](https://github.com/iovisor/bpftrace) 值得一试。

# Ubuntu22.04无法从外部拖文件进去

修改`/etc/gdm3/custom.conf`，删掉WaylandEnable=false这一行最开始的#号，重启即可。

# vmware Ubuntu会慢慢变卡问题

虚拟机打开`Intel VT-x/EPT`虚拟化。

若显示不支持虚拟化：[此平台不支持虚拟化 Intel-VT-x/EPT 或 AMD-V/RVI\_好奇的菜鸟的博客-CSDN博客](https://blog.csdn.net/qq_29752857/article/details/131409452)

vmware19已解决此问题。
# 版本号标准

[语义版本号](https://semver.org/)格式：`主版本号.次版本号.补丁号`。规则：

- 如果新的版本没有改变 API，请将补丁号递增；
- 如果您添加了 API 并且该改动是向后兼容的，请将次版本号递增；
- 如果您修改了 API 但是它并不向后兼容，请将主版本号递增。

>使用依赖管理系统的时候，您可能会遇到锁文件（_lock files_）这一概念。锁文件列出了您当前每个依赖所对应的具体版本号。通常，您需要执行升级程序才能更新依赖的版本。这么做的原因有很多，例如避免不必要的重新编译、创建可复现的软件版本或禁止自动升级到最新版本（可能会包含 bug）。

# 测试术语

- 测试套件：所有测试的统称。
- 单元测试：一种“微型测试”，用于对某个封装的特性进行测试。
- 集成测试：一种“宏观测试”，针对系统的某一大部分进行，测试其不同的特性或组件是否能_协同_工作。
- 回归测试：一种实现特定模式的测试，用于保证之前引起问题的 bug 不会再次出现。
- 模拟（Mocking）: 使用一个假的实现来替换函数、模块或类型，屏蔽那些和测试不相关的内容。例如，您可能会“模拟网络连接” 或 “模拟硬盘”。

# windows 端口被占用
```bash
#查询占用某端口的进程
netstat -ano|findstr 8080

#杀死一个进程
taskkill /pid 31601 /f
```

# 一大截端口被占用且不可查

[【转】解决 Windows 10 端口被 Hyper-V 随机保留（占用）的问题 - SpringCore - 博客园](https://www.cnblogs.com/fanqisoft/p/17071121.html)

# 哈希模取质数

对于三列函数$H(k)=k\%P$，P往往取质数。

原因是，比如P为4的之后，假如数列中差距为2的数字，如5,7,9，都会分布在同一个子数列中，导致容易冲突；而此时差距为4也会有这种情况出现。但如果P取质数，比如5，则只有差距为5的数字才会出现这种情况。

即：与模数不互质的数可以放到的位置更少。




















