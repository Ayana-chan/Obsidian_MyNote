
# 环境

通过配置`~/.pip/pip.conf`来配置pip，包括源。
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


# Embeddable Python

[Embeddable Python在工程实践上的使用 | Soulter's Blog](https://blog.soulter.top/posts/embbed-python.html)

官网下载时选择 Embeddable Package 可以下载精简版python, 它可以集成到项目中.

默认没有pip, 需要[下载`get-pip.py`文件](https://bootstrap.pypa.io/get-pip.py), 放到python根目录, 然后运行`python get-pip.py`.

> [!info]
> windows默认会先使用当前工作目录下的可执行文件, 再去环境变量里面找. 使用`python -c "import sys; print(sys.executable)"`查看python目录, 或看`where python`的第一个.

`cd`到`Scripts`文件夹中才能运行`pip`.

要把`python38._pth`里的`import site`的注释给去掉, 才能让其正常工作.

要在`python38._pth`里面加上项目路径(没有import): `../MyProName`. 这指示了python搜索模块的根目录, 因此如果有很多程序入口的话, 要全都配置上.

项目里使用相对路径的话就容易在使用Embeddable Python的时候报错.

# 问题

## pip无法运行

安装了多个python环境的话, 运行pip时很有可能会报`Fatal error in launcher: Unable to create process using ...`, 此时可以通过**强制重装**此python的pip来解决:

```bash
python -m pip install --upgrade --force-reinstall pip
```



# 库

## pymc

pymc在py3出了之后改名pymc3, 但后来又改回pymc.

这里记录python 3.9的pymc3. 使用conda的话创建空环境即可:
```bash
conda create -n pymc3_env python=3.9
```

给一些版本降级, 防止报错`distutils.msvccompiler`找不到:
```bash
pip install setuptools==58.0.0 wheel==0.36.2
```

然后安装pymc3:
```bash
pip install pymc3
```


# uv

[Site Unreachable](https://blog.yasking.org/a/python-project-manager-uv.html)

uv是 cargo for python, 基于虚拟环境 + toml依赖管理.

为项目生成并激活虚拟环境:
```shell
uv venv myenv
source myenv/bin/activate
```

可以在生成环境的时候使用`--python`指定python版本.

使用`uv pip`即可使用pip的命令(虽然没有完全支持所有命令).

从toml生成`requirements.txt`:
```shell
uv pip compile pyproject.toml -o requirements.txt
```



















