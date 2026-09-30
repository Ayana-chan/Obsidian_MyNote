
# 安装

安装命令取决于系统、Python、硬件及所选版本；torchvision、torchaudio 按需要安装。PyTorch 自 2.6 起不再发布官方 Conda 包；新环境按[官方安装选择器](https://pytorch.org/get-started/locally/)生成命令。[2.6 发布说明](https://pytorch.org/blog/pytorch2-6/)（核验：2026-09-30）。

下面保留曾用的 CUDA 12.1 / Conda 历史环境命令：

```shell
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

# 使用

## 自动求导

[一文解释 PyTorch求导相关 (backward, autograd.grad)](https://zhuanlan.zhihu.com/p/279758736)

其实是求梯度向量，即求出所有偏导。

导数是运行时进行构建计算的。
### 普通求导

设$y = (x + 1)(x + 2)$，求$\frac{\partial y}{\partial x}$：

```python
# 让自变量进行追踪
import torch
x = torch.tensor(2., requires_grad=True)
# x = torch.tensor(2.).requires_grad_()

# 设立函数（框架会记录计算树）
a = torch.add(x, 1)
b = torch.add(x, 2)
y = torch.mul(a, b)

# 让函数调用backward来对自变量求梯度
y.backward()
# 梯度结果被记在x.grad里面
print(x.grad)
# Output: tensor(7.)
```

### 向量间变换关系求导

对向量输出，backward 计算的是给定上游向量的 **VJP**。令 $h(\vec x)=\sum_j y_j$，则：

$$\frac{\partial h}{\partial x_i}=\sum_j\frac{\partial y_j}{\partial x_i}.$$

这是一个梯度向量，不能把它写成把所有对角偏导相加的标量。逐元素关系 $y_i=g(x_i)$ 是特例，此时第 $i$ 个分量就是 $g'(x_i)$。

```python
import torch
x = torch.arange(4., requires_grad=True)
y = x * x
# 等价于 y.backward(torch.ones_like(y))
y.sum().backward()
print(x.grad)
# tensor([0., 2., 4., 6.])
```

梯度会累积；重复训练步骤需清空 .grad。下面 detach 例子接着使用这里的 x。
[核对：autograd](https://docs.pytorch.org/tutorials/beginner/basics/autograd_tutorial.html)。

### 分离求导

假设`y`是`x`的函数，而`z`是`y`和`x`的函数。当计算`z`对`x`的梯度时，可能会要求此时将`y`视作常数。对y使用`detach`可以生成一个不传播求导的函数值u，使用u作为自变量的函数在进行求导时会将u视作常数：

```python
x.grad.zero_()
y = x * x
u = y.detach()
z = u * x

z.sum().backward()
x.grad == u
# Output: tensor([True, True, True, True])
```

可以在y上调用backward使其正常对x求导：

```python
x.grad.zero_()
y.sum().backward()
x.grad == 2 * x
# Output: tensor([True, True, True, True])
```






