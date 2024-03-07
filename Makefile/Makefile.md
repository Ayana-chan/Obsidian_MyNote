

# 基础

`make`是最常用的**构建系统**之一。

`make -j?`可以自动获取CPU核数进行并发编译。

执行`make`时，它会去参考当前目录下名为`Makefile`的文件。所有构建目标、相关依赖和规则都需要在该文件中定义，它看上去是这样的：

```makefile
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

冒号左侧的是**构建目标**，冒号右侧的是构建它所需的**依赖**。缩进的部分是从依赖构建目标时需要用到的一段**程序**。

若`make`不带参数，则默认构建第一条。以目标名为参数就可以构建该目标：`make plot-data.png`。

规则中的 `%` 是一种模式，它会匹配其左右两侧相同的字符串。例如，如果目标是 `plot-foo.png`， `make` 会去寻找 `foo.dat` 和 `plot.py` 作为依赖。

make会以目标为单位检查之前的构建是否因其依赖改变而需要被更新。如果一个目标的依赖都没变，则不会对该目标做任何事。

```
$#  是传给脚本的参数个数
$ 0 是脚本本身的名字
$ 1 是传递给该shell脚本的第一个参数
$ 2 是传递给该shell的脚本的第二哥参数
$@  是传给脚本的所有参数的列表，代表目标文件（target）
$*  是以一个单字符串显示所有向脚本传递的参数，与位置变量不同，参数可超过9个
$$  是脚本运行的当前进程ID号
$?  是显示最后命令的退出状态，0表示没有错误，其他表示有错误
$<  表示第一个依赖文件  
$^  代表所有的依赖文件（components）
@   这个符串通常用在“规则”行中，表示不显示命令本身，而只显示它的结果
```

```
$(MAKECMDGOALS) 参数传来的目标名，无目标时为空
```
# 函数


调用函数：

```makefile
#函数名和参数之间加空格
#多个参数之间加逗号
$(<function> <arguments>) 
#或 
${<function> <arguments>}

#例如
$(subst t,e,maktfilt)
```

`wildcard`函数获取所有文件名匹配了**某一个**参数的模式的文件。

```makefile
$(wildcard <PATTERN...>)

#返回make工作下的所有.cpp以及.c文件 
$(wildcard *.cpp *.c)
```

`patsubst`函数将符合pattern模式的text中的所有字符串（用空格分隔）都替换为replacement。可以使用`%`匹配任意长度的字符串，也可以将其写回到replacement当中去。

```makefile
$(patsubst <pattern>,<replacement>,<text>)

#把字符串“x.c.c bar.c”符合模式%.c的单词替换成%.o，返回“x.c.o bar.o”
$(patsubst %.c,%.o,x.c.c bar.c)
```

# 技巧

## 获取Makefile文件路径

由于默认使用的路径是执行make命令的路径，因此有些相对路径可能会出问题。用一下方法获取Makefile所在路径：

```Makefile
mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))  
DIR_ROOT := $(dir $(mkfile_path))
```

# 其他

## Windows

安装好mingw环境后，就可以使用mingw32-make来跑Makefile。












