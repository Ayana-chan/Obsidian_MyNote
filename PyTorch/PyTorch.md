

# 使用

## 自动求导

[一文解释 PyTorch求导相关 (backward, autograd.grad)](https://zhuanlan.zhihu.com/p/279758736)

设$y = (x + 1)(x + 2)$，求$\frac{\partial y}{\partial x}$：

```python
# 让自变量进行追踪
x = torch.tensor(2., requires_grad=True)
# x = torch.tensor(2.).requires_grad_()

# 设立函数（框架会记录计算树）
a = torch.add(x, 1)
b = torch.add(x, 2)
y = torch.mul(a, b)

# 让函数调用backward来对自变量求导
y.backward()
print(x.grad)
# Output: tensor(7.)
```












