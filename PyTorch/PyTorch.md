

# 使用

## 自动求导

[一文解释 PyTorch求导相关 (backward, autograd.grad)](https://zhuanlan.zhihu.com/p/279758736)

其实是求梯度向量，即求出所有偏导。

导数是运行时进行构建计算的。
### 普通求导

设$y = (x + 1)(x + 2)$，求$\frac{\partial y}{\partial x}$：

```python
# 让自变量进行追踪
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

假如$y_i = g(x_i)$，求标量函数$g$相对其自变量的梯度，但是代码里面往往只存了向量$\vec{x}$和$\vec{y}$，无法进行标量函数的求导；又或者，此时要求的就是偏导数之和。此时（用偏导数表示）应当计算：

$$\sum\frac{\partial y_i}{\partial x_i} = \frac{\partial \sum y_i}{\partial x_i}$$

即，令$h(\vec x) = \sum y_i$，求$\nabla h(\vec{x})$。

```python
# 对非标量调用backward需要传入一个gradient参数，该参数指定微分函数关于self的梯度。
# 本例只想求偏导数的和，所以传递一个1的梯度是合适的
x.grad.zero_()
y = x * x
# 等价于y.backward(torch.ones(len(x)))
y.sum().backward()
x.grad
# Output: tensor([0., 2., 4., 6.])
```

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






