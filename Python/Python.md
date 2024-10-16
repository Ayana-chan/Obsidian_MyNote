
# 基本使用

## 大对象原地操作内存

```python
# 重新分配Z的内存
X = X + Y
# 在X上原地操作
X[:] = X + Y
```


# 具体场景

## 读写csv

[2.2. 数据预处理 — 动手学深度学习 2.0.0 documentation](https://zh.d2l.ai/chapter_preliminaries/pandas.html)

以逗号分隔数据，用换行符来换行，以满足csv的格式。

```python
import os

os.makedirs(os.path.join('..', 'data'), exist_ok=True)
data_file = os.path.join('..', 'data', 'house_tiny.csv')
with open(data_file, 'w') as f:
    f.write('NumRooms,Alley,Price\n')  # 列名
    f.write('NA,Pave,127500\n')  # 每行表示一个数据样本
    f.write('2,NA,106000\n')
    f.write('4,NA,178100\n')
    f.write('NA,NA,140000\n')
```

读csv可以使用pandas库。

```python
import pandas as pd

data = pd.read_csv(data_file)
print(data)
```




